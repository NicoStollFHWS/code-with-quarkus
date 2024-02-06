
# Questions

## Nenne ein alternatives enterprise framework, um Geschäftsanwendungen zu entwickeln:

- Spring
- Quarkus
- Micronaut

## Nenne einen open-source application server und einen kommerziellen server, der den Jakarta EE standard vollständig implementiert

| open source            | kommerziell                            |
|------------------------|----------------------------------------|
| WildFly (früher JBoss) | IBM Websphere Application Server (WAS) |
| Payara Server          | Oracle WebLogic Server                 |
| Open Libery            |                                        |

## Welche Firma steht hinter dem open-source application server Wildfly?

Red Hat

## Welche Konzepte eines application server basieren auf dem Hollywood-Prinzip? Nenne ein paar Buzzwords.

"Don't Call Us, We'll Call You"

Inversion of Control (IoC) : Dependencies werden von einem Container oder 
Framework bereitgestellt und in die Komponenten injiziert

## Nennen Sie eine Möglichkeit, wie einem servlet container mitgeteilt werden kann, dass es sich bei einer Java Klasse um ein Servlet handelt

Servlet: Java-Klasse, die auf der Serverseite läuft und dazu dient,
Webanfragen zu verarbeiten und dynamische Webinhalte zu erzeugen. Es 
ist ein grundlegender Baustein in Java-basierten Webanwendungen
und implementiert das Servlet-Interface. Servlets arbeiten auf Ebene
von HTTP-Anfragen (GET, POST, PUT, DELETE).

```java
import jakarta.validation.Valid;
import jakarta.ws.rs.POST;
import jakarta.ws.rs.Path;
import jakarta.ws.rs.Produces;

@Path("students")
public class StudentResource {

    @Inject
    StudentService studentService;

    @POST
    @Produces(MediaType.APPLICATION_JSON)
    public StudentDTO createStudent(@Valid StudentCreateDTO student) {
        return studentService.persist(student);
    }

    //...
}
```

## Sie sollen einen Logger schreiben, der jede Http-Anfrage an den Server protokolliert. Mit welchem Konzept lässt sich dies sehr einfach in einem servlet container realisieren?

Filter

## Wie kann ich sichergehen, dass die Anfrage von einem angemeldeten User kommt?

Um sicherzugehen, dass die Anfrage von einem angemeldeten User kommt, muss
man ihn authentifizieren. Dafür kommen verschiedene Möglichkeiten infrage, z.B.
- Basic Authentication
- Bearer Token
- JWT (JSON Web Token) → hier gezeigt

```json
{
  "alg": "HS256",
  "typ": "JWT"
},
{
  "sub": "1234567890",
  "name": "John Doe",
  "iat": 1516239022
},
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  [your-256-bit-secret]
) 
```

Um sicherzugehen, dass dieser vor jeder Anfrage geprüft wird, kann man z.B.
einen Filter einsetzen, sodass jeder HTTP-Request diesen durchlaufen muss.

```java
@WebFilter("/*")
public class PerformanceFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {

        // filter logic goes here
        
       }

}
```

## Was ist die Grundidee hinter JPA?

Das Grundkonzept hinter der JPA (Java Persistence API) ist das Konzept
des **ORM** (Object Relational Mapping). Es ermöglicht Java-Entwicklern,
relationale Daten als Java-Objekte zu behandeln und umgekehrt.

JPA bietet eine Abstraktionsebene über JDBC (Java Database Connectivity).

## Mit welcher Annotation bilden sie eine 1:N Relation in JPA ab?

Die Annotation sind im folgenden Beispiel dargestellt:
- 1:N = @OneToMany
- N:1 = @ManyToOne

```java
import jakarta.persistence.Entity;
import lombok.Getter;
import lombok.Setter;
import jakarta.persistence.OneToMany;
import jakarta.persistence.ManyToOne;

@Entity
@Getter
@Setter
public class Student {

    //attributes
    
    @OneToMany
    @JoinColumn(name = "adress_id")
    private List<Address> addresses;
    
    @ManyToOne(cascase = CascadeType.ALL)
    private DegreeProgram degreeProgram;
}

```
## Gegeben ist eine Webanwendung exam, die Informationen rund um Klausuren verwaltet. In der Anwendung existiert die Entität Studenten mit folgenden Attributen: Name, Geburtsdatum. Wie muss die Entity Klasse hierfür aussehen?

Wichtig ist hier die Nutzung von ZonedDateTime, da dieses direkt in die Datenbank
gemappt werden kann.

```java
import java.time.ZonedDateTime;
import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.Id;
import jakarta.validation.constraints.NotBlank;
import lombok.Getter;
import lombok.Setter;

@Entity
@Getter
@Setter
public class Student {

    @Id
    @GeneratedValue
    private Long id;

    @NotBlank
    private String name;

    private ZonedDateTime geburtsdatum;
}
```

## Nutzung eines Entity Beans (vervollständigen einer Klasse):

```java
import jakarta.enterprise.context.ApplicationScoped;
import jakarta.persistence.EntityManager;
import jakarta.persistence.PersistenceContext;
import jakarta.transaction.Transactional;

@ApplicationScoped
public class StudentService {

    @PersistenceContext
    EntityManager em;
    
    public Student getStudent(Long id) {
        return em.find(Student.class, id);
    }
    
    public void deleteStudent(Long id) {
        Student student = getStudent(id);
        if(student != null) {
            em.remove(student);
        }
    }
    
    @Transactional
    public void addStudent(Student newStudent) {
        return em.persist(student);
    }

    @Transactional
    public Student updateStudent(Long id, Student newStudent) {
        if (id != student.getId()) {
            throw new Exception();
        }
        return em.merge(student);
    }
    
}
```
Transactional bei Post, bsp. logservice, der immer persistiert, egal was passiert (entityManager nennen)

## Mit welcher Annotation wird ein Objekt von CDI injiziert?

```java
@ApplicationScoped
public class StudentService {
    //...
}

@Path("students")
public class StudentResource {

    @Inject
    StudentService studentService;
    
    //...
}

```
## Sie sollen ein Objekt vor dem injezieren mit speziellen Werten initialisieren. Hierzu schreiben sie eine dafür geeignete Methode + Annotation.

Warum? Initialisierung von Objekten z.B. basierend auf:
- App Configuration
- spezifischen Request

```java
public class CdiQuarkusProducer {

    @Produces
    @DefaultBean
    @RequestScoped
    public CdiQuarkusData dataProducer() {
        System.out.println("CdiQuarkusProducer#dataProducer");

        CdiQuarkusData data = new CdiQuarkusData();
        data.setName("Based on Quarkus Producer");

        return data;
    }
}

```
## Benennen sie die Funktion von Events zum Absenden und Empfangen.

| synchron  | asynchron      |
|-----------|----------------|
 | fire()    | fireAsync()    |
 | observe() | observeAsync() |

```java

@Qualifier
@Target({ElementType.PARAMETER, ElementType.FIELD, ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface NewStudentEvent {
}

@ApplicationScoped
public class StudentService {
    
    // ...
    
    @Inject
    @NewStudentEvent
    Event<StudentDTO> newStundentEvent;
    

    @Transactional
    public StudentDTO persist(StudentCreateDTO student) {

        // ...
        StudentDTO dtoPayload = newStudent.toDTO();

        // ...

        newStundentEvent.fireAsync(dtoPayload);
    }
}

public class EventExample {
    
    public void eventNotification(@ObservesAsync @NewStudentEvent StudentDTO student) {

        System.out.println("Event Example: " + student.getFirstName() + " " + student.getEmail());

    }

}

```

## Webservice und die URL des Endpunkts angeben

Annotationen: @Path @GET

```java
@Path("students")
public class StudentResource {

    @Inject
    StudentService studentService;

    @POST
    public StudentDTO createStudent(@Valid StudentCreateDTO student) {
        return studentService.persist(student);
    }

    @GET
    @Path("/{studentNumber}")
    public StudentDTO getStudentByStudentNumber(@PathParam("studentNumber") String studentNumber) {
        return studentService.findByStudentNumber(studentNumber);
    }
}
```
## Microprofile

### mit welcher Annotation kann man die Config API ansprechen?

@ConfigProperty(name = "some.property")

### Wie müssen die Annotations aussehen, damit der Container der Member Variable msg die Environment Variable message zuweist

@Inject
@Config Property

### Microprofile healthcheck

Health 4.0

### Microprofile API für lange Laufzeiten

FaultTolerance 4.0

### Welche APIs kennen Sie?

| MicroProfile API           | Funktionalität                                                                                                                      |
|----------------------------|-------------------------------------------------------------------------------------------------------------------------------------|
| Telemetry 1.1              | Bietet APIs für die Überwachung von Microservices durch Sammeln von Metriken und Tracing-Informationen.                             |
| OpenAPI 3.1                | Ermöglicht die Dokumentation von RESTful APIs in einem standardisierten Format.                                                     |
| Rest Client 3.0            | Bietet eine typsichere Möglichkeit, RESTful-Dienste zu konsumieren.                                                                 |
| Config 3.1                 | Ermöglicht die externe Konfiguration von Anwendungen und Microservices.                                                             |
| FaultTolerance 4.0         | Bietet Funktionen wie Retry, Circuit Breaker, Timeout, Bulkhead und Fallback zur Verbesserung der Fehlertoleranz von Microservices. |
| Metrics 5.1                | Ermöglicht die Sammlung und Auswertung von Metriken aus Microservices.                                                              |
| JWT Authentication 2.1     | Ermöglicht die Authentifizierung und Autorisierung von Benutzern durch Verwendung von JSON Web Tokens (JWT).                        |
| Health 4.0                 | Bietet APIs für die Implementierung von Health Checks in Microservices.                                                             |
| Jakarta EE 10 Core Profile | Ein Profil von Jakarta EE, das eine Reihe von Technologien für die Erstellung von Enterprise Java-Anwendungen definiert.            |

### Nennen Sie eine API, mit der Rest API dokumentiert werden können

Open API 3.1

## Für was steht CI? Für was steht CD?

CI (Continuos Integration): Änderungen  werden entwickelt, geprüft und
in einem gemeinsamen Repository zusammengefügt.

CD (Continuous Delivery): Änderungen werden automatisch auf Bugs getestet
und in ein Repository hochgeladen, von wo aus sie vom OPs-Team in einer 
Live-Produktivumgebung bereitgestellt werden können.

https://www.redhat.com/de/topics/devops/what-is-ci-cd

## Wie heißt die Datei mit Bauanleitung für das Docker Image?

Docker file

## Nennen Sie zwei Vorteile, wenn sie ihre Java Anwendung als docker native in docker runtimes wie kubernetes betreiben

- weniger memory
- schnellere startup time

## wie können bei Runtime annotierte Klassen gefunden werden?

Reflection API

## Postgres in Docker. Wie können lokale settings gespeichert werden?

```dockerfile
version: '3.8'

services:
  db:
    image: postgres:12.17
    restart: always
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres123
      POSTGRES_DB: thws
    ports:
      - "5432:5432"
      
   # ...

```
docker compose

## Wie kann man application.properties überschreiben/setzen?

Properties können in der application.properties gesetzt werden:
```properties
thws.location = Würzburg
```

Setzen eines Wertes mit MicroProfile @ConfigProperty:
```java
@Path("mp")
public class MpResource {

    @Inject
    @ConfigProperty(name = "thws.location")
    String location;
}
```
## Prepared statements

Named Queries sind Teil der JPA Spezifikation. Sie werden zum Start der Anwendung
kompiliert und sind daher 'vorbereitet'. Sie werden für statische, wiederverwendbare
Abfragen verwendet, die in der gesamten Anwendung gleich bleiben.

```java
@Entity
@NamedQueries({
        @NamedQuery(name = Student.FIND_BY_STUDENT_NUMBER, query = "SELECT s FROM Student s WHERE s.studentNumber = :"
                + Student.PARAM_STUDENT_NUMBER),
        @NamedQuery(name = Student.FIND_ALL, query = "SELECT s FROM Student s")
})

public class Student {
    public static final String FIND_ALL = "Student.findAll";
    public static final String FIND_BY_STUDENT_NUMBER = "Student.findByStudentNumber";
    public static final String PARAM_STUDENT_NUMBER = "studentNumber";

    // ...
}

@ApplicationScoped
public class StudentService {

    @PersistenceContext
    EntityManager em;

    public StudentDTO findByStudentNumber(String studentNumber) {
        return em.createNamedQuery(Student.FIND_BY_STUDENT_NUMBER, Student.class)
                .setParameter(Student.PARAM_STUDENT_NUMBER, studentNumber)
                .getSingleResult()
                .toDTO();
    }
    
    // ...
}
```
@queries

## Unterschied von merge und persist im EM

- merge liefert etwas zurück
- persist geht nur auf neuen Objekten

## build-tool für die paketierung von Java Anwendungen

Maven

## BCE-Pattern (boundary-entity-control-pattern)

| Klasse   | Rolle                                                                                                                                                                                                                              |
|----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Entity   | Die Entity-Klassen repräsentieren die Geschäftsobjekte oder Datenobjekte in der Anwendung. Sie enthalten in der Regel Geschäftsdaten und die damit verbundenen Geschäftsregeln.                                                    |
| Boundary | Die Boundary-Klassen stellen die Schnittstelle zur Außenwelt dar. Sie sind verantwortlich für die Interaktion mit Benutzern oder externen Systemen. In einer Webanwendung könnten dies beispielsweise die Controller-Klassen sein. |
| Control  | Die Control-Klassen orchestrieren die Anwendungslogik und steuern den Ablauf der Anwendung. Sie koordinieren die Interaktion zwischen den Boundary- und Entity-Klassen.                                                            |

## wofür steht DTO und wofür braucht man Sie?

Data Transfer Objekt

Wird häufig verwendet um:
- Daten über Netzwerke zu senden: DTOs können serialisiert werden
- Daten an Nutzer senden - Präsentation in geeignetem Format und Datensparsamkeit
- Schutz vor Senden sensibler Informationen druch Weglassen sicherheitsrelevanter Informationen

## bei der Kommunikation von Microservices via REST empfängt der Konsument oft JSON Daten (ohne Typisierung). Welche Microprofile API ermöglicht eine sichere Typisierung?

Rest Client

## Wie schreibt man eine Custom Exception?

```java
public class ThwsValidationException extends ValidationException {

    @Getter
    ThwsValidationDTO payload;

    public ThwsValidationException(ThwsValidationDTO payload) {
        this.payload = payload;
    }
}

```

Exceptions können gefangen werden und so eine Response an den User weitegegeben werden:
```java
@Provider
public class ThwsValidationExceptionMapper implements ExceptionMapper<ThwsValidationException> {

    @Override
    public Response toResponse(ThwsValidationException exception) {
        return Response.status(Status.BAD_REQUEST).entity(exception.getPayload()).build();
    }

}
```
## DTO und entityBeans: Wie kann man mappen?

```java
public class Car {
 
    private String make;
    private int numberOfSeats;
    private CarType type;
 
    //constructor, getters, setters etc.
}

public class CarDto {

    private String make;
    private int seatCount;
    private String type;

    //constructor, getters, setters etc.
}

@Mapper
public interface CarMapper {

    CarMapper INSTANCE = Mappers.getMapper( CarMapper.class );

    @Mapping(source = "numberOfSeats", target = "seatCount")
    CarDto carToCarDto(Car car);
}
```

mapstruct: https://mapstruct.org/


