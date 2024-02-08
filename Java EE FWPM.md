# Allgemein
## Specification

- Java EE für Enterprise Application -> implementing business and presentation logic
- Java Community Process (JCP), definiert Java Specification Requests (JSR)
- Wichitg: Java EE ist nur Spezifikation, Implementierung von Application Server
- Geschichte: J2EE 1.2  1.4, JavaEE 5 - 8, Jakarta EE 8-10 (vor JavaEE 5 war alles umständlich)
## Application Server
#### Containerkonzept
Referenzimplementierung: Glassfish Application Server

| Open Source | Kommerziell |
| ---- | ---- |
| Glassfish - Oracle | Oracle AS |
| JBoss AS - Redhat | Websphere - IBM |
| Wildfly - Redhat | SAP NetWeaver |

Keine Application Server:
- Apache HTTP
- Tomcat
- Jetty
- Node.js
- nginx

#### Paketierung
- JAR = Java Archive, EJB (Jakarta Enterprise Beans) Archives
- WAR = Web Archive (Servlets, JSP, HTML etc)
- EAR = Enterprise Archive (Jar + War + EJB-Jar), aber unnötig

#### Server
Inversion of Control = Hollywood Prinzip: Don't call us, we call you
-> Application Server called meinen Business Code

## Servlets
- Basis aller Java EE Frontend Technologien
- ersetzt Common Gateway Interface (CGI), weil besser, schneller, sicherer
- Seit Java EE 5: Convention over Configuration
- Lebenszyklus: init() - service() - destroy()
Webfilter (wie Interceptor, für Logging, Header Manipulation etc)
Weblistener (hören auf bestimmte Ereignisse, wie Session expired etc)

#### Servlet Code
```java
@WebServlet("/endpoint")
public class MyServlet extends HttpServlet {
	@Override
	public void doGet(HttpServletRequest request, HttpServletResponse response) {
		// important methods:
		String lang = reqest.getHeader("Accept-Language");
		String query = request.getQueryString();
		Map<String, String[]> parameterMap = request.getParameterMap(); // query parameter
		BufferedReader bodyReader = request.getReader();

		response.getWriter().println("respponse body");
		response.setStatus(200);
		// for redirection -> 302 Found
		response.sendRedirect("http://example.com");
		// for forwarding (server side)
		reponse.getRequestDispatcher("/api/hello").forward(reqest, response);
	}
}
```
#### Webfilter Code
```java
@WebFilter("/")
public class MyFilter implements Filter {
	@Override  
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
		// Beispiel Request blockieren, wenn xy
		if (blockRequest) {
			return;
		}
		else {
			chain.doFilter(request, response);
		}
	}
}
```

## Buildmanagement
Tools:
- Maven
- Gradle
- Jenkins

Maven life cycle (convention over configuration):
validate - compile - test - package - integration-test - verify - install - deploy

## Docker
- Containerizing Applications

### Befehle
- `docker ps` zeigt laufende Docker Prozesse
- `docker build -f FILE_PATH -t TAG PATH` baut Dockerfile im PATH mit dem Tag TAG
- `docker run -d -p host:container -v /host/path:/opt/container/path IMAGE` führt IMAGE detached aus mit dem angegeben Port Mapping und bind mounted ein volume
- `docker cp src/path CONTAINERID:/opt/container/path`
### Dockerfile
```dockerfile
FROM repository:tag
ADD /src/ /dest/ # add files from source to destination
USER root # act as root user
RUN mkdir /folder # execute shell commands
WORKDIR /dest # set /dest as working directory -> . is now /dest
EXPOSE 8080 # makes port 8080 available for listening
```

### Docker Compose File
```yml
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
  
  app:  
    image: quarkus/students-jvm  
    environment:  
      quarkus.datasource.jdbc.url: jdbc:postgresql://db:5432/thws  
      thws.location: "within the cloud"  
    ports:  
      - "9090:8080"
```

# Web Services

- Web Services sind für M2M-Communication
- jeder Webservice hat einen Uniform Resource Identifier (URI)
- hat maschinenlesbare Schnittstellenbeschreibung
### JAX-WS
=  Jakarta XML Web Services
Standardprotokoll: SOAP = Simple Object Access Protocol
Annotation: `@WebService`

### JAX-RS
= Jakarta RESTful Web Services
Standardprotokoll: HTTP = Hypertext-Transfer-Protocol

Jersey als Referenzimplementierung
Alternative: JBoss RESTEasy, Restlet

#### HTTP Methods
GET, PUT, POST, DELETE, HEAD, OPTIONS
#### Application Config Code
```java
@ApplicationPath("api")  
public class ApplicationConfig extends Application {  }
```

#### Rest Resource Code
```java
@Path("hello")  
public class RestResource {  
    @GET
    @Path("/{id}")
    @Produces(MediaType.APPLICATION_JSON)  
    public String hello(
	    @PathParam("id") String id,
	    @QueryParam("query") String query,
	    @DefaultValue("20") int limit
	) {  
        return "Hello";
    }  

	@POST
    @Consumes(MediaType.APPLICATION_JSON)  
    public hello(Data someData) {  // jakarta validation
        return "Hello";
    }
}
```
#### Validation
JAX-RS erlaubt Beans zu validieren:

```java
public class StudentCreateDTO {  
    @NotEmpty  
    private String firstName;  
    @NotEmpty  
    private String lastName;  
    @NotEmpty  
    @Email    
    private String privateEmail;  
}

public class StudentResource() {
	@POST
	@Consumes(MediaType.APPLICATION_JSON)
	public void createStudent(@Valid StudentCreateDTO createDTO) {
		
	}
}
```

#### Exception Mapping
```java
@AllArgsConstructor
@Getter
public class InvalidExcpetion extends ValidationExecption {
	private final int statusCode;
	private final String payload;
}

public class InvalidExcpetionMapper implements ExceptionMapper<InvalidExcpetion> {
	@Override
	public Repsonse toResponse(InvalidExcpetion e) {
		return Response.status(e.getStatusCode()).entity(e.getPayload()).build();
	}
}
 
public class StudentService {
	public void update(Long id, Student student) {  
	    if (id != student.getId()) {  
	        throw new InvalidExcpetion(400, "path id != student.id");
	    }  
	    // ...
	}
}
```
# Quarkus
= Ein Java Stack für Cloud Native Java Enterprise Applications
designed für Containers, Kubernetes, GraalVM
unterstützt:
- JAX-RS (JBoss RESTEasy)
- JPA ORM (Hibernate),
- CDI (ArC)
- MicroProfile
Alternativen:
- Spring
- Micronaut
- Node.js
- Vert.x
- Javalin

# Java Persistence API (JPA)
Referenzimplementierung: eclipseLink
Alternative: Hibernate

Früher:
JDBC = Java Database Connectivity
JDBC in Wildfly:

```xml
<datasource jta="true" jndi-name="java:jboss/datasources/FHWS-DS"
pool-name="FHWS-DS" enabled="true" use-java-context="true">
	<connection-url>
		jdbc:h2:tcp://localhost/D:/fhws/servers/database/fhws-db
	</connection-url>
	<driver>h2</driver>
	<security>
		<user-name>sa</user-name>
		<password>sa</password>
	</security>
</datasource>
```

## Object Relational Mapping (ORM)
ersetzt JDBC
### Application Properties
in `application.properties`:
```properties
quarkus.datasource.db-kind = postgresql 
quarkus.datasource.username = postgres  
quarkus.datasource.password = postgres123 
quarkus.datasource.jdbc.url = jdbc:postgresql://localhost:5432/mydb
```

### Entity
Beispiel von `Entity`s und ihren Annotations:
```java
@Entity  
@NamedQueries({  // definiert queries neben standart find(), siehe EnitityManager Beispiel für Anwendung
        @NamedQuery(name = "Student.findByStudentNumber", query = "SELECT s FROM Student s WHERE s.studentNumber = :studentNumber"),  
        @NamedQuery(name = "Student.findAll", query = "SELECT s FROM Student s")  
})  
public class Student {  
    @Id  
    @GeneratedValue
    private Long id;
    @NotBlank // mind. 1 non-whitespace character
    private String studentNumber;
    
    private ZonedDateTime created;  
	private ZonedDateTime updated;  
  
	@ManyToOne(cascade = CascadeType.ALL)  // n Students to 1 DegreeProgram
	@JoinColumn(name = "degreeprogram_id") // optional, Hibernate mappt auch ohne
	private DegreeProgram degreeProgram;

	@ManyToMany(mappedBy = "students")
    private Set<Course> courses = new HashSet<>();
	
	@PreUpdate
	@PrePersist  
	void setTimeStamps() {  
	    if (created == null)  
	        created = ZonedDateTime.now();  
	    updated = ZonedDateTime.now();  
	}
}

@Enitity
public class DegreeProgram {
	@Id
	@GeneratedValue
	Long id;
	
	@OneToMany(mappedBy = "degreeprogram")  // 1 DegreeProgram to n Students
	//oder statt mappedBy: @JoinColumn(name = "degreeprogram_id")  
	private List<Student> students;
}

@Entity
public class Course {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;

    @ManyToMany
    @JoinTable(  // JoinTable nur bei "besitzender Seite"
        name = "course_student", 
        joinColumns = @JoinColumn(name = "course_id"), 
        inverseJoinColumns = @JoinColumn(name = "student_id")
    )
    private Set<Student> students = new HashSet<>();
}

```
### DTOs
= Data Transfer Objects
dienen, um nicht Entity Klassen an Schnittstellen zu exposen
macht Schnittstelle sauberer und getrennt von DB Modell

`@JsonBTransient` unnötig

Beispiel `StudentDTO` und `StudentCreateDTO`
```java
@Data  
@Builder  //lombock
public class StudentDTO {  
    private String studentNumber;  
    private String firstName;  
    private String lastName;
}

@Data  
public class StudentCreateDTO {  
    @NotEmpty  // jakarta validation: @Valid StudentCreateDTO will ensure validation
    private String firstName;  
    @NotEmpty  
    private String lastName;  
    @NotEmpty  
    @Email
    private String privateEmail;
    private String degreeProgramKey
}
```

### Entity Manager

`EntityManager` Methoden:
```java
interface EntityManager<T> {
	void persist(T t); // persitiert Entity in DB (nach Transaktionscommit)
	T find(Class<T> clazz, Object key); // findet Entity anhand von ID
	T merge(T t); // merged aktuelle Entity mit DB Kontext und returnt merged Objekt
	void remove(T t); // entfertn Entity aus der DB (nach Transaktionscommit)
}
```
#### Status einer JPA-Entity
- New (nach der Erstellung, vor persist): noch nicht der DB bekannt
- Managed (nach persist(), find() oder merge()): wird mit DB bei nächstem Commit gesynct
- Detached (nach detach() oder nach commit / rollback): nicht mehr verbunden 
- Removed (nach remove() im Zustand Managed): wird bei commit aus der DB entfernt und danach detached.
#### Code Beispiel `EntityManager`
```java
public class StudentService {
	@PersistenceContext
	EntityManager<Student> em;
	@Inject
	DegreeProgramService dps;
	
	@Transactional  // macht Methode tranaktional
	public StudentDTO persist(StudentCreateDTO createDTO) {
		Student s = new Student();
		// map StudentCreateDTO to Student
		s.setDegreeProgram(dps.findByKey(createDTO.getDegreeProgramKey()));
		// other logic
		em.persist(s);
		return s;
	}
	
	public List<StudentDTO> list() {  
	    return em.createNamedQuery("Student.findAll", Student.class)  
	            .getResultStream().map(Student::toDTO)  
	            .toList();  
	}
}
```

### Transactional
macht Methoden transaktional, d.h vor dem Aufruf wird eine Transaktion begonnen und erst nach dem Aufruf wird die Transaktion commitet (Ist ein Interceptor).

```java
@Transactional // default value ist TxType.REQUIRED -> erstellt eine neue Transaktion, wenn noch keine vorhanden ist.
public void persit(Data data) {}

@Transactional(TxType.REQUIRES_NEW) // erstellt immer neue Transaktion, unabhängig von bereits laufenden Transaktionen
public void persit(Data data) {}
```
# Context and Dependency Injection (CDI)
Referenzimplementierung: Weld
Alternative: OpenWebBeans, ArC

Vorteile:
- Erweiterbarkeit
- Wartbarkeit
- Testbarkeit
#### Begriffe
Bean: Von CDI gemanagtes Objekt
Context: bindet Komponente (Bean) an bestimmten Lifecycle (bspw. Request vs Application Scope)
Dependency Injection (DI): Implementierung von Komponenten (Bean) können beim Deployment gewählt werden und werden automatisch injeziert.
--> Lose Kopplung

DI ist eine Anwendung von Inversion of Control auf Objektebene

#### Scopes
Normal Scopes (Injection Point bekommt Proxy Objekt, verlinken beim Aufruf):
- Application Scoped
- Session Scoped 
- ConversationScoped (über mehrere Seiten hinweg, aber nicht über gesamte Session)
- RequestScoped
Pseudo Scopes (Injection Point bekommt direkt Objekt über Producer-Methode):
- Singleton
- Dependent (hängt von übergeordneter Bean ab)

#### Code Beispiel #1 Simpel
```java
@ApplicationScoped // Producer-Methode: new StudentService()
public class StudentService {
	@PostConstruct
	void init() {
		// will be called after StudentService Constructor
	}
	// methods
}

public class InjectionExample {
	@Inject
	StudentService studentService;
}

```
#### Code Beispiel #2 Qualifier
```java

public interface Service {
	void serve();
}

@ApplicationScoped
@DefaultBean
public class ServiceImpl1 implements Service {
	public void serve() {
		System.out.println("Serve 1");
	}
}

@Qualifier
@Target({ElementType.PARAMETER, ElementType.FIELD, ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@interface Service2Qualifier {
}

@ApplicationScoped
@Service2Qualifier
public class ServiceImpl2 implements Service {
	public void serve() {
		System.out.println("Serve 2");
	}
}

public class InjectionExample {
	@Inject
	Service service1; // will be ServiceImpl1
	@Inject
	@Service2Qualifier
	Service service2; // will be ServiceImpl2

	public void doStuff() {
		service1.serve(); // prints "Serve 1"
		serivce2.serve(); // prints "Serve 2"
	}
}
```
#### Beispiel #3 Producer Method
```java
public class ProducedService implements Service {
	private final String someProperty;
	public Service(String someProperty) {
		this.someProperty = someProperty;
	}
	public void serve() {
		System.out.println("Serve " + someProperty);
	}
}

public class ProducerExample {
	@Produces // note: jakarta.enterprise.inject.Produces != jakarta.ws.rs.Produces 
	@Named("ProducedService")
	@ApplicationScoped
	public Service produceService() { // method may accepts InjectionPoint
		String someProperty = // read properties or whatever...
		return new Service(someProperty)
	}
}

public class InjectionExample {
	@Inject
	@Named("ProducedService")
	Service service; // will be ProducedService with the loaded Property

	public void doStuff() {
		service.serve(); // prints "Servce " and the value of someProperty
	}
}

```

### Interceptor (Aspektorientierte Programmierung)
nützlich für Logging, Security, `@Transactional` (ist ein Interceptor, in dem tx.begin() und tx.commit aufgerufen wird)

```java
@InterceptorBinding
@Target(...)
@Retention(RUNTIME)
public @interface Logged {
} 

@Interceptor
@Logged
public class LoggingInterceptor {
	@AroundInvoke
	public Object log(InvocationContext ctx) {
		Logger logger = Logger.getLogger(ctx.getTarget().getClass().getName());
		logger.info("log before call");
		Object result = ctx.proceed();
		logger.info("log after call");
		return result:
	}
}

@ApplicationScoped
@Logged
public class StudentService {
	public void doStuff() {
		// logs will be called before and after this method invocation
	}
}
```

### Stereotype
verbindet mehrere Annotations zu einer

```java
@Stereotype
@ApplicationScoped
@Transactional
@Target({TYPE, METHOD, FIELD})
@Retention(RUNTIME)
public @interface Service {
}

@Service // hat jetzt alle Annotationen von oben
public class StudentService {
}
```

### Events
Events können nach dem Observer Pattern beobachtet werden.

```java
public class SomeEvent {
	// event information
}

public class SomeService {
	@Inject
	Event<SomeEvent> someEvent

	public void serve() {
		someEvent.fire(new SomeEvent(...));
		// oder fireAsync(...) für asynchrone Events
	}
}

public class EventObserver {
	public void doStuffOnEvent(@Observes SomeEvent someEvent) { // oder @ObservesAsync
		// ...
	}
}
```

Um unterschiedliche SomeEvents zu unterscheiden, können `@Qualifier`s verwendet werden;
```java
@Qualifier
public @interface StudentCreatedEvent {
}

public class StudentService() {
	@Inject
	@StudentCreatedEvent
	Event<Student> studentCreatedEvent;

	public void createStudent(StudentCreateDTO createDTO) {
		Student createdStudent = createDTO.toEnitity();
		// persist(createdStudent);
		studentCreatedEvent.fire(createdStudent);
	}
}

public class StudentCreatedEventObserver {
	public void doStuffOnEvent(@Observes @StudentCreatedEvent Student createdStudent) { 
		// ...
	}
}
```

### Reflection API
Javas API, um Metadaten über Objekte zu verwalten.
Kann benutzt werden, um CDI mit Annotations durchzuführen (Quarkus unter der Haube):
```java
public @interface SomeAnnotation {
}

public class SomeService {
	// ...
}

public class InjectionManager {
	public void injectInObject(Object o) {
		Class clazz = o.getClass();
		for (Field f : clazz.getDeclaredFields()) {
			for (Annotation a : f.getAnnotations()) {
				if (a.equals(SomeAnnotation.class)) {
					o.set(f, new SomeService());
				}
			}
		}
	}
}

```

# MicroProfile
ist Optimierung / Erweiterung von Jakarta EE, designed für Microservices und Cloud Native

| MicroProfile API | Funktionalität |
| ---- | ---- |
| Telemetry 1.1 | Bietet APIs für die Überwachung von Microservices durch Sammeln von Metriken und Tracing-Informationen. |
| OpenAPI 3.1 | Ermöglicht die Dokumentation von RESTful APIs in einem standardisierten Format. |
| Rest Client 3.0 | Bietet eine typsichere Möglichkeit, RESTful-Dienste zu konsumieren. |
| Config 3.1 | Ermöglicht die externe Konfiguration von Anwendungen und Microservices. |
| FaultTolerance 4.0 | Bietet Funktionen wie Retry, Circuit Breaker, Timeout, Bulkhead und Fallback zur Verbesserung der Fehlertoleranz von Microservices. |
| Metrics 5.1 | Ermöglicht die Sammlung und Auswertung von Metriken aus Microservices. |
| JWT Authentication 2.1 | Ermöglicht die Authentifizierung und Autorisierung von Benutzern durch Verwendung von JSON Web Tokens (JWT). |
| Health 4.0 | Bietet APIs für die Implementierung von Health Checks in Microservices. |
| Jakarta EE 10 Core Profile | Ein Profil von Jakarta EE, das eine Reihe von Technologien für die Erstellung von Enterprise Java-Anwendungen definiert. |

#### Code Beispiel Config 
```java
@Config("some.property")
String someProperty; // will be the value of some.property in application.properties
```
#### Code Beispiel Metrics
```java
@Counted(name = "performedChecks", description = "Aufruf-Häufigkeit")  
@Timed(name = "checksTimer", description = "Aufruf-Dauer", unit = MetricUnits.MILLISECONDS)  
public void method() { ... }
```
#### Code Beispiel Fault Tolerance
```java
@Retry(maxRetries = 3, delay = 2000)  
@Fallback(fallbackMethod = "defaultCourse")  
public String getFirstCourse() {  
	Client c = ClientBuilder.newBuilder().build();  // JAX-RS-Client
	JsonArray resultArray = c.target("http://localhost:9090/")  
			.path("courses")  
			.request()  
			.get(JsonArray.class);
}
```
#### Code Beispiel Health Check
```java
@Liveness  
@Readiness  
public class MyHealthCheck implements HealthCheck {  
    @Override  
    public HealthCheckResponse call() {  
        return HealthCheckResponse.up("successful-check");  
    }  
}
```
## Testing in Quarkus

```java
@QuarkusTest  
public class MyTest {  
    @Test  
    public void testHelloEndpoint() {  
        given()  
          .when().get("/hello")  
          .then()  
             .statusCode(Status.OK.getStatusCode())  
             .body(is("Hello THWS from Würzburg"));  
    }
}
```