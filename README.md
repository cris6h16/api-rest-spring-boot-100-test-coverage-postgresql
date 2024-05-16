# api-rest-spring-boot-basic-auth-TDD--notes-app

Implementing a Semantic API REST in Spring Boot, using basic auth applying TDD, this is a little Note Application which
will use PostgresSQL (password is used encrypted for everything).

## 1. DEFINING THE PROJECT

### 1.1. User Case

- ANONYMOUS CAN:
    - create an account


- USER CAN:
    - update his account data
    - create a note
    - update a note
    - "delete" a note
    - list all his notes
    - See a note
    - delete his account


- ADMIN CAN:
    - list all users

### 1.2. Some considerations

- If the user isn't the owner of the note, then for him the note doesn't exist.
- The user can't hard delete a note or his account, only soft delete.
- The user can't modify audit fields.
- remember that `PUT` update or create.
- user can only get his profile information
- we'll return body in something goes wrong for those which isn't necessary return a response body if was successful

### 1.3 Endpoints

| URI                                                                            | HTTP METHOD | SUCCESS          | DESC             | Authority    |
|--------------------------------------------------------------------------------|-------------|------------------|------------------|--------------|
| `/api/users/{id}`                                                              | `GET`       | `200 OK`         | _see a user_     | `ROLE_USER`  |
| `/api/users`                                                                   | `POST`      | `201 CREATED`    | _create a user_  |              |
| `/api/users`                                                                   | `PATCH`     | `204 NO CONTENT` | _update a user_  | `ROLE_USER`  |         
| `/api/users`                                                                   | `DELETE`    | `204 NO CONTENT` | _delete a user_  | `ROLE_USER`  |
| `/api/notes`                                                                   | `POST`      | `201 CREATED`    | _create a note_  | `ROLE_USER`  |
| `/api/notes`<br/>- `page=<1>`<br/>- `size=<2>`<br/>- `sort=<create_at>, <asc>` | `GET`       | `200 OK`         | _list all notes_ | `ROLE_USER`  |
| `/api/notes/{id}`                                                              | `GET`       | `200 OK`         | _see a note_     | `ROLE_USER`  |
| `/api/notes/{id}`                                                              | `PUT`       | `204 NO CONTENT` | _update a note_  | `ROLE_USER`  |
| `/api/notes/{id}`                                                              | `DELETE`    | `204 NO CONTENT` | _delete a note_  | `ROLE_USER`  |
| `/api/users`<br/>- `page=<1>`<br/>- `size=<2>`<br/>- `sort=<create_at>, <asc>` | `GET`       | `200 OK`         | _list all users_ | `ROLE_ADMIN` |

### 1.4 Entities

- User
    - id
    - username (max=20 & min=1)
    - email
    - password (min=8)
    - created_at
    - updated_at
    - ~~deleted_at~~ (User won't delete softly )
    - roles
    - notes


- Note
    - id
    - title (MAx=255)
    - content
    - ~~created_at~~ (I won't add it for use `PUT`)
    - updated_at
    - ~~deleted_at~~ (Note won't delete softly )


- Role
    - id
    - name (ERole)

## 2 API REST

### 2.1 CREATE a USER

| URI          | HTTP METHOD | SUCCESS       |
|--------------|-------------|---------------|
| `/api/users` | `POST`      | `201 CREATED` | 

| FAIL                                                                                                                                                    | RESPONSE          |
|---------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------|
| - _Username already exists_<br/>- _Email already exists_                                                                                                | `409 Conflict`    |
| - _Password must be at least 8 characters_<br/>- _Email is invalid_<br/>- _Email, Username and Password are required_<br/>- _Username mustn't be blank_ | `400 Bad Request` |

### 2.2 UPDATE a USER

| URI          | HTTP METHOD | SUCCESS          |
|--------------|-------------|------------------|
| `/api/users` | `PATCH`     | `204 NO CONTENT` |

| FAIL                                                                          | RESPONSE            |
|-------------------------------------------------------------------------------|---------------------|
| - _Username already exists_<br/>- _Email already exists_                      | `409 Conflict`      |
| - _You need to be authenticated to perform this action_                       | `401 Unauthorized`  |
| - _You aren't the owner of this id_ ( whether this Id exists/doesn't exist  ) | `403 Forbidden`     |
| - _Password must be at least 8 characters_<br/>- _Email is invalid_           | `400 Bad Request`   |
| ~~- _User not found_~~ -> _"replace" by -_ _You aren't the owner of this id_  | ~~`404 Not Found`~~ |

### 2.3 GET a USER

| URI               | HTTP METHOD | SUCCESS  |
|-------------------|-------------|----------|
| `/api/users/{id}` | `GET`       | `200 OK` |

| FAIL                                                                            | RESPONSE            |
|---------------------------------------------------------------------------------|---------------------|
| - _You need to be authenticated to perform this action_                         | `401 Unauthorized`  |
| - _You aren't the owner of this ID_   ( whether this Id exists/doesn't exist  ) | `403 Forbidden`     |
| ~~- _User not found_~~                                                          | ~~`404 Not Found`~~ |

<hr>  

### 2.3 CREATE a NOTE

| URI          | HTTP METHOD | SUCCESS       |
|--------------|-------------|---------------|
| `/api/notes` | `POST`      | `201 CREATED` |

| FAIL                                                                                                    | RESPONSE           |
|---------------------------------------------------------------------------------------------------------|--------------------|
| - _Title is required \*\*\*if only spaces or null\*\*\*_<br/>- _Title must be less than 255 characters_ | `400 Bad Request`  |
| - _You need to be authenticated to perform this action_                                                 | `401 Unauthorized` |

### 2.4 LIST all NOTES

| URI                                                                            | HTTP METHOD | SUCCESS  |
|--------------------------------------------------------------------------------|-------------|----------|
| `/api/notes`<br/>- `page=<1>`<br/>- `size=<2>`<br/>- `sort=<create_at>, <asc>` | `GET`       | `200 OK` |

| FAIL                                                    | RESPONSE           |
|---------------------------------------------------------|--------------------|
| - _You need to be authenticated to perform this action_ | `401 Unauthorized` |

### 2.5 SEE a NOTE

| URI               | HTTP METHOD | SUCCESS  |
|-------------------|-------------|----------|
| `/api/notes/{id}` | `GET`       | `200 OK` |

| FAIL                                                    | RESPONSE           |
|---------------------------------------------------------|--------------------|
| - _You need to be authenticated to perform this action_ | `401 Unauthorized` |
| - _Note not found_                                      | `404 Not Found`    |

### 2.6 UPDATE/CREATE a NOTE (pass the id)

| URI              | HTTP METHOD | SUCCESS          |
|------------------|-------------|------------------|
| `api/notes/{id}` | `PUT`       | `204 NO CONTENT` |

| FAIL                                                                     | RESPONSE           |
|--------------------------------------------------------------------------|--------------------|
| - _Title is required_<br/>- _Title must be between 1 and 255 characters_ | `400 Bad Request`  |`
| - _You need to be authenticated to perform this action_                  | `401 Unauthorized` |

### 2.7 DELETE a NOTE

| URI              | HTTP METHOD | SUCCESS          |
|------------------|-------------|------------------|
| `api/notes/{id}` | `DELETE`    | `204 NO CONTENT` |

| FAIL                                                    | RESPONSE           |
|---------------------------------------------------------|--------------------|
| - _You need to be authenticated to perform this action_ | `401 Unauthorized` |
| - _Note not found_                                      | `404 Not Found`    |

<hr>

### 2.8 LIST all USERS

| URI                                                                            | HTTP METHOD | SUCCESS  |
|--------------------------------------------------------------------------------|-------------|----------|
| `/api/users`<br/>- `page=<1>`<br/>- `size=<2>`<br/>- `sort=<create_at>, <asc>` | `GET`       | `200 OK` |

| FAIL                                                    | RESPONSE           |
|---------------------------------------------------------|--------------------|
| - _You need to be authenticated to perform this action_ | `401 Unauthorized` |
| - _You need to be an admin to perform this action_      | `403 Forbidden`    |

### 2.9 DELETE a USER

| URI          | HTTP METHOD | SUCCESS          |
|--------------|-------------|------------------|
| `/api/users` | `DELETE`    | `204 NO CONTENT` |

| FAIL                                                    | RESPONSE           |
|---------------------------------------------------------|--------------------|
| - _You need to be authenticated to perform this action_ | `401 Unauthorized` |
| - _You aren't the owner of this ID_                     | `403 Forbidden`    |
| - _User not found_ *** _if you let multi sessions_ ***  | `404 Not Found`    |

#

## 3. TDD

TDD defines what the system should do, not how it should do it.   
Help us to ensure that the code is working as expected, and it's easier to maintain due that "all"
behavior is covered by tests.
any change made in the future will trigger a test failure.

Developing with TDD ensure the coverage of "all" possible code which are coverable, in this case mainly the Controllers,
is also necessary write test for boundary cases in the `@Service` (just an example).

![img_3.png](img_3.png)

- One of my coverage in a `@Service` class

![img_11.png](img_11.png)  
![img_9.png](img_9.png)  
![img_8.png](img_8.png)  
![img_7.png](img_7.png)  
![img_6.png](img_6.png)  
![img_5.png](img_5.png)  
![img_4.png](img_4.png)

I recommend you run yourself the tests to see the coverage

## Things that can be forgotten

The behavior that can be forgotten is the behavior which is considered boundary cases, for example:

1. parameter receives Longs, but the user passes a String.
2. parameter was passed null.  
   Although my code will be handled automatically due that I used `@Transactional` with `Rollback` for every exception.
   the real problem here can be the response body, which may not be descriptive enough for the user.   
   but this can be solved checking yourself the boundary cases and throw exceptions with a descriptive message,   
   also can be handled by  `@<RestController>Advice` which can send a generic response, and in the front simply say some
   like "An error occurred, please try again later", but this doesn't be so frequently, only for cases
   like errors in the database, transactions, etc. these mentioned can also provide a more descriptive message writing
   explicitly

## Time taken (UTC-5)

**PD**: This isn't classified very well, but I'll try to do it better in the future.   
also take into account that I'm already have an idea of what the api will do. Only was capture my general idea in
specific points (for defining & things like that).

### README.md

1. Setup project

| Date       | Defined | Start | End   |
|------------|---------|-------|-------|
| 2024-04-28 |         | 11:50 | 12:21 |

2. [DEFINING THE PROJECT](#1-defining-the-project)

| Date       | Defined                                                       | Start | End   |
|------------|---------------------------------------------------------------|-------|-------|
| 2024-04-28 | [User Case & Endpoints](#11-user-case)                        | 12:47 | 13:12 |
| 2024-04-28 | [Entities](#14-entities) & Corrections(endpoints, error code) | 13:30 | 14:00 |

3. [API REST](#2-API-REST)

| Date       | Defined                                                                             | Start           | End             |
|------------|-------------------------------------------------------------------------------------|-----------------|-----------------|
| 2024-04-28 | [USER CREATION](#21-create-a-user)                                                  | 12:24           | 12:47           |
| 2024-04-28 | [UPDATED A USER](#22-update-a-user)                                                 | 13:15           | 13:30           |
| 2024-04-28 | [OPERATIONS WITH NOTES](#23-create-a-note) & Corrections & last review of responses | 14:00<br/>17:22 | 14:30<br/>17:50 |
| 2024-04-28 | git conflict                                                                        | 14:30           | 14:41           |
| 2024-04-28 | Review & Corrections in Response Codes                                              | 15:50           | 16:05           |
| 2024-05-07 | [DELETE A USER](#29-delete-a-user)                                                  | 21:40           | 21:50           |

**∞**. Additional

| Date       | Defined              | Start | End   |
|------------|----------------------|-------|-------|
| 2024-04-28 | ENHANCED readability | 21:35 | 21:45 |

### IMplementation

4. Beginning the Implementation

| Date                                     | Defined                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     | Start                                                               | End                              |
|------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------|----------------------------------|
| 2024-04-28                               | [shouldCreateAUser()](src/test/java/org/cris6h16/apirestspringboot/Controllers/UserControllerTest.java)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     | 21:55                                                               | 22:44                            |
| 2024-04-28                               | Correct(IMPL) datasource, Impl Entities, DTO, .YAML, etc<br/>basically the second startup                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | 22:44                                                               | 00:15                            |
| 2024-04-28                               | Analyzing a error that I was([@JsonFormat](src/main/java/org/cris6h16/apirestspringboot/DTOs/CreateUserDTO.java))                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           | 00:15                                                               | 00:30                            |
| 2024-04-29<br/>2024-04-30                | Implementing(Repository, Entities, Security, etc)    --> [shouldCreateAUser()](src/test/java/org/cris6h16/apirestspringboot/Controllers/UserControllerTest.java)<br/>Test passed                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            | 19:13<br/>14:15                                                     | 20:33<br/>15:09                  |
| 2024-04-30                               | [shouldNotCreateAUser_usernameAlreadyExists()](src/test/java/org/cris6h16/apirestspringboot/Controllers/UserControllerTest.java) <br/> & Test passed                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        | 15:40                                                               | 16:10                            |
| 2024-04-30                               | [shouldNotCreateAUser_EmailAlreadyExists()](src/test/java/org/cris6h16/apirestspringboot/Controllers/UserControllerTest.java) <br/> & Test passed                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           | 16:15                                                               | 16:35                            |
| 2024-04-30                               | [shouldNotCreateAUser_PasswordTooShort()](src/test/java/org/cris6h16/apirestspringboot/Controllers/UserControllerTest.java) <br/> & Test passed <br/> & Analyzing why always is greater that 8 <br/>(is saved encrypted we need verify length before)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       | 16:40                                                               | 16:57                            |
| 2024-04-30                               | [shouldNotCreateAUser_EmailIsInvalid()](src/test/java/org/cris6h16/apirestspringboot/Controllers/UserControllerTest.java) <br/> & Test passed                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               | 17:00                                                               | 17:10                            |
| 2024-05-01                               | [shouldNotCreateAUser_UsernameIsNullOrEmpty()](src/test/java/org/cris6h16/apirestspringboot/Controllers/UserControllerTest.java)<br/>[shouldNotCreateAUser_PasswordTooShort()](src/test/java/org/cris6h16/apirestspringboot/Controllers/UserControllerTest.java)<br/>[shouldNotCreateAUser_PasswordIsNullOrEmpty](src/test/java/org/cris6h16/apirestspringboot/Controllers/UserControllerTest.java)<br/>[shouldNotCreateAUser_EmailIsNullOrEmpty](src/test/java/org/cris6h16/apirestspringboot/Controllers/UserControllerTest.java)<br/> Investigating & analyzing why some data constraints aren't  respected<br/> and find a way to handle better the exceptions (response body) <br/>Correct all responses (standardize the errors responses)<br/> [see that](#things-that-caused-me-troubles)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           | 16:55                                                               | 20:10                            | 
| 2024-05-03                               | Add/Improve docs/comments                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | 17:25                                                               | 17:48                            |
|                                          |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |                                                                     |                                  |
| 2024-05-03<br/>2024-05-04                | Tests added(UPDATE USER): <br/>[shouldUpdateUsername()](src/test/java/org/cris6h16/apirestspringboot/Controllers/UserControllerTest.java)<br/>[shouldUpdateEmail()]((src/test/java/org/cris6h16/apirestspringboot/Controllers/UserControllerTest.java))<br/>[shouldUpdatePassword()](src/test/java/org/cris6h16/apirestspringboot/Controllers/UserControllerTest.java)<br/>[shouldUpdateUsernameEmailPassword()](src/test/java/org/cris6h16/apirestspringboot/Controllers/UserControllerTest.java)<br/>[shouldBeGreaterUpdatedAt()](src/test/java/org/cris6h16/apirestspringboot/Controllers/UserControllerTest.java)<br/>[updateShouldNotChange_DeletedAtCreatedAtRolesNotes()](src/test/java/org/cris6h16/apirestspringboot/Controllers/UserControllerTest.java)<br/>[shouldNotUpdateUsernameAlreadyExists()](src/test/java/org/cris6h16/apirestspringboot/Controllers/UserControllerTest.java)<br/>[shouldNotUpdateEmailAlreadyExists()](src/test/java/org/cris6h16/apirestspringboot/Controllers/UserControllerTest.java)<br/>[shouldNotUpdateYouNeedToBeAuthenticated()](src/test/java/org/cris6h16/apirestspringboot/Controllers/UserControllerTest.java)<br/>[shouldNotUpdateYouCannotUpdateOtherUserAccount()](src/test/java/org/cris6h16/apirestspringboot/Controllers/UserControllerTest.java)<br/>[shouldNotUpdateUserNotFound()](src/test/java/org/cris6h16/apirestspringboot/Controllers/UserControllerTest.java)<br/>[shouldNotUpdateEmailIsInvalid()](src/test/java/org/cris6h16/apirestspringboot/Controllers/UserControllerTest.java)<br/>[shouldNotUpdatePasswordTooShort()](src/test/java/org/cris6h16/apirestspringboot/Controllers/UserControllerTest.java)                                                                                                                                                                                                                                                                                                                                                                                                                                                            | 18:06<br/> 02:20                                                    | 20:30<br/> 03:20                 |
| 2024-05-04                               | Implementing the above test && Tests passed                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | 10:05<br/>16:22                                                     | 12:20<br/>18:17                  |
| 2024-05-04<br/><br/><br/><br/>2024-05-05 | Tests added(GET USER):<br/>[shouldGetAUserById()](src/test/java/org/cris6h16/apirestspringboot/Controllers/UserControllerTest.java)<br/>[shouldNotGetIsNotAuthenticated()](src/test/java/org/cris6h16/apirestspringboot/Controllers/UserControllerTest.java)<br/>[shouldNotGetUserYouAreNotTheOwner()](src/test/java/org/cris6h16/apirestspringboot/Controllers/UserControllerTest.java)_<br/> Implementing & Passing the tests                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             | 21:32<br/><br/><br/><br/>07:51                                      | 21:54<br/><br/><br/><br/>8:17    |
| 2024-05-05 <br/><br/>                    | corrected security issues in [MyAuthorizationService](src/main/java/org/cris6h16/apirestspringboot/Service/CustomAuthHandler/MyAuthorizationService.java) <br/>Adding  Test(CREATE A NOTE):<br/> [shouldCreateANote()](src/test/java/org/cris6h16/apirestspringboot/Controllers/NoteControllerTest.java)<br/>[shouldNotCreateANoteTitleIsNull()](src/test/java/org/cris6h16/apirestspringboot/Controllers/NoteControllerTest.java)<br/>[shouldNotCreateANoteTitleIsBlank()](src/test/java/org/cris6h16/apirestspringboot/Controllers/NoteControllerTest.java)<br/>[shouldNotCreateANoteTitleLengthIsGreaterThan255()](src/test/java/org/cris6h16/apirestspringboot/Controllers/NoteControllerTest.java)<br/>[shouldNotCreateANoteMustBeAuthenticated()](src/test/java/org/cris6h16/apirestspringboot/Controllers/NoteControllerTest.java)<br/>Corrected the entities relationship<br/>added [NoteEntities.txt](src/test/resources/NotesEntities.txt) -> are serialized in JSON -> for the next tests<br/>Tests turn to green                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | 8:20<br/>11:37<br>13:42<br/>+ aprox: 1 hour I forget take last time | 10:20<br/>12:05<br>14:40<br><br> |
| 2024-05-05<br/>2024-05-06                | GetAllNotes Tests(Pageable):<br/>[shouldListAllNotesIsPageable()](src/test/java/org/cris6h16/apirestspringboot/Controllers/NoteControllerTest.java)<br>[~~shouldNotListAllNotesIsPageable()~~](src/test/java/org/cris6h16/apirestspringboot/Controllers/NoteControllerTest.java)<br/>[shouldNotListAllNotesIsNotAuthenticated()](src/test/java/org/cris6h16/apirestspringboot/Controllers/NoteControllerTest.java)<br/>[URIWithNoParamsShouldListInDefaultPageableConfiguration()](src/test/java/org/cris6h16/apirestspringboot/Controllers/NoteControllerTest.java)<br>refactor [shouldListAllNotesIsPageable_ASC_DESC](src/test/java/org/cris6h16/apirestspringboot/Controllers/NoteControllerTest.java)<br/>&& Tests passed                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | 19:25<br/>14:46<br/>18:00                                           | 20:06<br/>16:58<br/>18:35        |
| 2024-05-06                               | GetANote Test added:<br/>[shouldGetANote()](src/test/java/org/cris6h16/apirestspringboot/Controllers/NoteControllerTest.java)<br/>[shouldNotGetANoteIsNotAuthenticated()](src/test/java/org/cris6h16/apirestspringboot/Controllers/NoteControllerTest.java)<br/>[shouldNotGetANoteIsNotFound()](src/test/java/org/cris6h16/apirestspringboot/Controllers/NoteControllerTest.java)<br/>[shouldNotGetNoteIsNotFoundBecauseIsNotTheOwner()](src/test/java/org/cris6h16/apirestspringboot/Controllers/NoteControllerTest.java)<br/>&& Test passed                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               | 19:35                                                               | 20:35                            |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             
| 2024-05-07                               | UPDATE/CREATE a NOTE (PUT) Tests Added:<br/>[shouldCreateANotePUT()](src/test/java/org/cris6h16/apirestspringboot/Controllers/NoteControllerTest.java)<br/>[shouldNotCreateANotePUTTitleIsNull()](src/test/java/org/cris6h16/apirestspringboot/Controllers/NoteControllerTest.java)<br/>[shouldNotCreateANotePUTTitleIsBlank()](src/test/java/org/cris6h16/apirestspringboot/Controllers/NoteControllerTest.java)<br>[shouldNotCreateANotePUTTitleLengthIsGreaterThan255()](src/test/java/org/cris6h16/apirestspringboot/Controllers/NoteControllerTest.java)<br/>[shouldNotCreateANotePUTMustBeAuthenticated()](src/test/java/org/cris6h16/apirestspringboot/Controllers/NoteControllerTest.java)<br/>[shouldReplaceANotePUT()](src/test/java/org/cris6h16/apirestspringboot/Controllers/NoteControllerTest.java)<br/>[shouldNotReplaceANotePUTTitleIsNull()](src/test/java/org/cris6h16/apirestspringboot/Controllers/NoteControllerTest.java)<br/>[shouldNotReplaceANotePUTTitleIsBlank()](src/test/java/org/cris6h16/apirestspringboot/Controllers/NoteControllerTest.java)<br/>[shouldNotReplaceANotePUTTitleLengthIsGreaterThan255()](src/test/java/org/cris6h16/apirestspringboot/Controllers/NoteControllerTest.java)<br/>[shouldNotReplaceANotePUTMustBeAuthenticated()](src/test/java/org/cris6h16/apirestspringboot/Controllers/NoteControllerTest.java)<br/>[shouldDeleteANote()](src/test/java/org/cris6h16/apirestspringboot/Controllers/NoteControllerTest.java)<br/>[deleteANoteShouldNotDeleteTheUser()](src/test/java/org/cris6h16/apirestspringboot/Controllers/NoteControllerTest.java)<br/>[shouldNotDeleteANoteIsNotAuthenticated()](src/test/java/org/cris6h16/apirestspringboot/Controllers/NoteControllerTest.java)<br/>[shouldNotDeleteANoteIsNotFound()](src/test/java/org/cris6h16/apirestspringboot/Controllers/NoteControllerTest.java)<br/>[shouldNotDeleteANoteIsNotTheOwnerNotFound()](src/test/java/org/cris6h16/apirestspringboot/Controllers/NoteControllerTest.java)<br/>&& Test passed<br/>**PD**: All test was passed in the 1st try(except 1) that's not surprising when your logic is very modular | 16:45<br/>19:04                                                     | 17:15<br/>20:45                  |
| 2024-05-08                               | DELETE A USER tests, impl:<br/>[shouldDeleteAUser()](src/test/java/org/cris6h16/apirestspringboot/Controllers/UserControllerTest.java)<br/>[shouldNotDeleteAUserIsNotAuthenticated()](src/test/java/org/cris6h16/apirestspringboot/Controllers/UserControllerTest.java)<br/>[shouldNotDeleteAUserYouAreNotTheOwner()](src/test/java/org/cris6h16/apirestspringboot/Controllers/UserControllerTest.java)<br/>[shouldDeleteAllNotesWhenDeleteAUser()](src/test/java/org/cris6h16/apirestspringboot/Controllers/UserControllerTest.java)<br/>&& Tests passed                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | 21:38                                                               | 22:20                            |
| 2024-05-08<br/>2024-05-09                | List all users(`ADMIN`) tests, impl:<br/>[with27Users1AdminInDB](src/test/java/org/cris6h16/apirestspringboot/Controllers/UserControllerTest.java):<br/>[shouldListAllUsersInPagesIsPageable()](src/test/java/org/cris6h16/apirestspringboot/Controllers/UserControllerTest.java)<br/>[shouldNotListAllUserIsPageable()](src/test/java/org/cris6h16/apirestspringboot/Controllers/UserControllerTest.java)<br/>[shouldListAllUsersDefaultConfigNotUrlParams()](src/test/java/org/cris6h16/apirestspringboot/Controllers/UserControllerTest.java)<br/>[shouldNotListAllUserIsNotAdmin()](src/test/java/org/cris6h16/apirestspringboot/Controllers/UserControllerTest.java)<br/>&& Tests passed                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               | 22:26<br>14:40                                                      | 23:22<br>17:15                   |
| 2024-05-13/14                            | Refactor<br/>Hardcode related to User & Auth was centralized                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | around 5 hours                                                      |                                  |
| 2024-05-14                               | Refactor<br/>Hardcode related to Note was centralized                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       | around 2 hours                                                      |                                  |
| 2024-05-15                               | doing the TODOs(<br>[shouldNotCreateAUser_UsernameIsTooLong(), loginBadCredentials(), passingAStrInsteadOfANumberShouldReturnBadRequest()]()<br>)<br>_I won't take the time, the majority of the time, Im writing the README_                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |                                                                     |                                  |

- User tests:

![img.png](img.png)    
![img_1.png](img_1.png)

- Note tests:

![img_2.png](img_2.png)

### Some Questions that you probably have

1. How is taken the time ?

> The time is taken since I start to do it & when I finish it. NOT is only the time of coding or something like that.

2. Why are there very short/large cuts(time) between them ?

> I'm a student, I have to do other things(Homework, study for lessons, etc), and I'm a son, my parents unexpectedly
> call me to do something or else they get angry.   
> also I'm a human, I need to eat, sleep, etc...

## THINGS THAT PROBABLY CAN CAUSE TROUBLES TO YOU

1. Validations from different packages

> if you validate with a field like `@Length(min = .., max = .., message = ..)` which come
> from `org.hibernate.validator.constraints` when you want to **Validate** from a parameter for example (`@Valid`) which
> come from `<javax>.validation.constraints` you validation will be ignored(If you are validating an element that you
> won't persist) specially if you passed a null/empty value.
> if
> you want to validate the said use
`@Size` & `@NotBlank` (in `@Size` null elements are valid) from `<javax>.validation.constraints`; or directly
> use `@NotBlank`.

2. Standardize the responses

> One of the things that take additional time was that literally I had to guess the response body(this problem only if
> there was an exception)   
> I had been having 2 format of responses(and probably can easily be added more);
> 1. when I throw an `ResponseStatusException` or any of its subclasses from `@Service` for example.
> 2. when I handle exceptions from `@<RestController>Advice` which is different from the above.
> 3. possible increase the number of formats(if in each Exception handled in Advice I return a different format (very
     possible)).

> Did you see the problem? my way to solve it was make a common Exception response for
> all (`DataIntegrityViolationException`, `ConstraintViolationException`, `ResponseStatusException`, any threw in
> anywhere ) using a `Map` and `ObjectMapper` in `@<RestController>Advice`

2. `@EqualsAndHashCode`

> if you put `@EqualsAndHashCode` in your `@Entity` or any class, for a correct work you need to put this annotation
> in all contained into this class(classes which are User-defined).
> Example: if you have a class `User` which contains a class `Role` & `Note` you need to put `@EqualsAndHashCode` in all
> of them.

3. Verify the length of the password

> I was trying to reach the length fail(min) of the password, but I was always getting a length greater than 8.
> The problem was that I wasn't taking into account that the password is saved encrypted.
> Then i cannot use `@Length(min = 8, message = "Password ....")` because the password is encrypted & always will be
> greater.
> Due that I did the verification in the `@Service` directly from the DTO.
> Now is the first verification of all, if you pass all attributes null/empty, then the password length fail will be the
> response message.

4. `TestRestTemplate` uses `SimpleClientHttpRequestFactory`, which does not support `PATCH` requests.

> This can lead to test failures if your tests involve PATCH operations.
> To fix this, you can configure the `TestRestTemplate` to use `HttpComponentsClientHttpRequestFactory`:   
> from `org.apache.httpcomponents.client5:httpclient5:x.x.x` then:   
> `@Autowired`    
> `TestRestTemplate rt`   
> `rt.getRestTemplate().setRequestFactory(new HttpComponentsClientHttpRequestFactory());`

5. How a `@ResController` which returns a `ResponseEntity<Void>` can have a body if it fails ?

> The `ResponseEntity<Void>` is a response entity which doesn't have a body, but if something goes
> wrong(Exceptions like ConstraintViolationException, DataIntegrityViolationException, etc)
> then it can have a body with `Map<String, Object>` which contains the error message(s).
> Simply because any exception thrown will return its own `ResponseEntity<String>` or any that you want.

6. Something goes wrong internally, but you don't know the specific reason why it fails

> Simply turn on the debug mode in the properties file, and you'll probably can see the specific reason why it fails in
> the logs.


// TODO: write about the definition of formats for exceptions/"errors", handling in HTTP handled classes exceptions, and
when throw a
ResponseStatusException, and when return a custom response entity




<hr>
//TODO: write about how all can test the API (POSTGRESQL setup)