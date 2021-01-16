
```kotlin
// © 2021 sammidev
package com.sammidev
```
```kotlin
import org.springframework.boot.WebApplicationType.SERVLET
import org.springframework.boot.context.event.ApplicationReadyEvent
import org.springframework.data.r2dbc.core.DatabaseClient
import org.springframework.fu.kofu.application
import org.springframework.fu.kofu.r2dbc.r2dbcH2
import org.springframework.fu.kofu.webmvc.webMvc
import org.springframework.web.servlet.function.ServerRequest
import org.springframework.web.servlet.function.ServerResponse
import java.time.LocalDate
```

```kotlin
fun main(args: Array<String>) {
    application(SERVLET) {
		listener<ApplicationReadyEvent> {
			with(ref<StudentRepository>()) {
               initialize()
				insert(Student(1L, "Sammi","0000000000", "sammi@gmail.com","+00000" , LocalDate.of(2020, 10, 20)))
				insert(Student(2L, "Aditya","0000000001", "adit@gmail.com","+00001" , LocalDate.of(2020, 10, 20)))
				insert(Student(3L, "Ayatullah","0000000002", "ayat@gmail.com","+00002" , LocalDate.of(2020, 10, 20)))
				insert(Student(4L, "Gusnur","0000000003", "gusnur@gmail.com","+00003" , LocalDate.of(2020, 10, 20)))
				insert(Student(5L, "Rauf","0000000004", "rauf@gmail.com","+00004" , LocalDate.of(2020, 10, 20)))
				insert(Student(5L, "Dandi","0000000005", "dandi@gmail.com","+00005" , LocalDate.of(2020, 10, 20)))
			}
		}
		beans {
			bean<StudentRepository>()
			bean<StudentHandler>()
		}
		webMvc {
			router {
				"/student".nest {
					GET("/{id}", ref<StudentHandler>()::readOne)
					GET("/") { ref<StudentHandler>().readAll() }
				}
			}
			converters {
				jackson()
			}
		}
		r2dbcH2()
	}.run(args)
}
```

```kotlin
class StudentHandler(private val studentRepository: StudentRepository) {
	fun readAll() = ServerResponse.ok().body(studentRepository.findAll())
	fun readOne(request: ServerRequest) = ServerResponse.ok().body(studentRepository.findById(request.pathVariable("id").toLong()))
}
```

```kotlin
class Student(
    val id: Long,
    val name: String,
    val nim: String,
    val email: String,
    val phone: String,
    val birthdate: LocalDate? = null)
```

```kotlin
class StudentRepository(private val client: DatabaseClient) {
	fun findAll() = client.select()
        .from(Student::class.java)
        .fetch().all()
        .collectList()
        .blockOptional()

	fun findById(id: Long) = client
        .execute("SELECT * FROM STUDENT WHERE ID = :id")
        .bind("id", id)
        .`as`(Student::class.java)
        .fetch()
        .one()
        .blockOptional()

	fun insert(student: Student) = client.insert()
        .into(Student::class.java)
        .table("STUDENT")
        .using(student)
        .then()
        .block()

	fun initialize() = client.execute(
	"""
    CREATE TABLE IF NOT EXISTS STUDENT (
        ID         BIGINT      NOT NULL AUTO_INCREMENT PRIMARY KEY,
        NAME       VARCHAR(50) NOT NULL,
        NIM        VARCHAR(10) NOT NULL UNIQUE,
        EMAIL      VARCHAR(50) NOT NULL UNIQUE,
        PHONE      VARCHAR(50) NOT NULL,
        BIRTHDATE  DATE
    );
	""")
        .then()
        .block()
}
```