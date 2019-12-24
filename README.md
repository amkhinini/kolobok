# kolobok
Java Annotation Processor for Spring. It contains a set of annotation that helps working with Spring easier. I am inspired by [Lombok](https://projectlombok.org/).

## @FindWithOptionalParams
Spring Data Repository interface may to include find methods. Something like 
```
@Repository
public interface PersonRepo extends PagingAndSortingRepository<Person, Long> {
    Iterable<Person> findByFirstNameAndLastNameAndCityId(String firstName, String lastName, Long cityId);
}
```
But if you want to find only by `cityId` and `lastName` you need to add new method
```
    Iterable<Person> findByLastNameAndCityId(String lastName, Long cityId);
```
This annotation allows to use original `findByFirstNameAndLastNameAndCityId` method with `null` values for params which should not included to search criteria.
```
@Repository
public interface PersonRepo extends PagingAndSortingRepository<Person, Long> {
    @FindWithOptionalParams
    Iterable<Person> findByFirstNameAndLastNameAndCityId(String firstName, String lastName, Long cityId);
}
```
So now you can use 
```
    Iterable<Person> persons = personRepo.findByFirstNameAndLastNameAndCityId(null, "Smith", 1L);
```
### How it works?
It generates all possible find methods for search params.
```
  Iterable<Person> cityId(Long cityId);
  Iterable<Person> lastName(String lastName);
  Iterable<Person> lastNameAndCityId(String lastName, Long cityId);
  Iterable<Person> firstName(String firstName);
  Iterable<Person> firstNameAndCityId(String firstName, Long cityId);
  Iterable<Person> firstNameAndLastName(String firstName, String lastName);
  Iterable<Person> firstNameAndLastNameAndCityId(String firstName, String lastName, Long cityId);
```
Also it generates default implementation of `findByFirstNameAndLastNameAndCityId` method that checks which params are null and calls corresponded method. 
```
    default Iterable<Person> findByFirstNameAndLastNameAndCityId(String firstName, String lastName, Long cityId) {
      if(firstName == null) {
        if(lastName == null) {
          if(cityId == null) {
            return findAll();
          } else {
            return cityId(cityId);
          }
        } else {
          if(cityId == null) {
            return lastName(lastName);
          } else {
            return lastNameAndCityId(lastName, cityId);
          }
        }
      } else {
        if(lastName == null) {
          if(cityId == null) {
            return firstName(firstName);
          } else {
            return firstNameAndCityId(firstName, cityId);
          }
        } else {
          if(cityId == null) {
            return firstNameAndLastName(firstName, lastName);
          } else {
            return firstNameAndLastNameAndCityId(firstName, lastName, cityId);
          }
        }
      }
    }

```
