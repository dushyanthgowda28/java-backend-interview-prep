# Spring Core — Topics for 7–8 Year Experienced Developer

## 1. Spring Framework Fundamentals
- Spring Framework architecture
- Spring modules
- Core Container
- Spring Core
- Spring Beans
- Spring Context
- Spring Expression Language (SpEL)
- Spring AOP
- Spring infrastructure overview
- Spring Framework vs Spring Boot

## 2. Inversion of Control (IoC)
- IoC concept
- IoC Container
- Dependency Injection (DI)
- Dependency Inversion Principle
- Benefits of IoC
- BeanFactory
- ApplicationContext
- BeanFactory vs ApplicationContext
- IoC container startup

## 3. Dependency Injection
- Constructor injection
- Setter injection
- Field injection
- Constructor vs setter vs field injection
- Required vs optional dependencies
- Immutable dependencies
- Dependency resolution
- Dependency injection internals

## 4. Spring Beans
- Bean definition
- Bean metadata
- Bean registration
- Bean naming
- Bean creation
- Bean lookup
- Bean instantiation
- Bean initialization
- Bean destruction
- getBean()
- Factory methods
- FactoryBean
- ObjectFactory
- ObjectProvider

## 5. Bean Scopes
- Singleton
- Prototype
- Request
- Session
- Application
- WebSocket
- Singleton scope internals
- Prototype scope internals
- Singleton vs prototype
- Scoped dependencies
- Scoped proxies
- proxyMode
- @Lookup method injection

## 6. Bean Lifecycle
- Bean lifecycle overview
- Instantiation
- Populate properties
- Aware interfaces
- BeanNameAware
- BeanFactoryAware
- ApplicationContextAware
- EnvironmentAware
- Before initialization
- BeanPostProcessor
- InstantiationAwareBeanPostProcessor
- Initialization callbacks
- @PostConstruct
- InitializingBean
- init-method
- After initialization
- Destruction callbacks
- @PreDestroy
- DisposableBean
- destroy-method
- Prototype destruction limitation

## 7. BeanPostProcessor
- BeanPostProcessor
- beforeInitialization
- afterInitialization
- Custom BeanPostProcessor
- Ordering of BeanPostProcessors
- InstantiationAwareBeanPostProcessor
- SmartInstantiationAwareBeanPostProcessor
- MergedBeanDefinitionPostProcessor
- AutowiredAnnotationBeanPostProcessor
- CommonAnnotationBeanPostProcessor

## 8. BeanFactoryPostProcessor
- BeanFactoryPostProcessor
- BeanDefinition modification
- PropertySourcesPlaceholderConfigurer
- Custom BeanFactoryPostProcessor
- BeanFactoryPostProcessor vs BeanPostProcessor

## 9. Dependency Resolution and Autowiring
- @Autowired
- @Qualifier
- @Primary
- @Resource
- @Inject
- Autowire byName
- Autowire byType
- Constructor autowiring
- Multiple candidate resolution
- Qualifier resolution
- Primary candidate
- Collection injection
- Map injection
- Optional dependencies
- Optional<T>
- ObjectProvider<T>
- Dependency resolution order
- @Autowired internals

## 10. Component Scanning
- Component scanning
- @ComponentScan
- Base packages
- ComponentScan filters
- Include filters
- Exclude filters
- TypeFilter
- AnnotationTypeFilter
- AssignableTypeFilter
- AspectJTypeFilter
- RegexPatternTypeFilter
- Custom filters
- Stereotype annotations
- @Component
- @Service
- @Repository
- @Controller

## 11. Configuration
- XML configuration
- Annotation-based configuration
- Java-based configuration
- @Configuration
- @Bean
- @Import
- @ImportResource
- @PropertySource
- Configuration classes
- Full @Configuration mode
- Lite @Configuration mode
- CGLIB enhancement
- @Bean method interception
- Configuration class processing

## 12. Bean Definition and Registration
- BeanDefinition
- BeanDefinitionRegistry
- GenericBeanDefinition
- RootBeanDefinition
- ChildBeanDefinition
- AnnotatedBeanDefinition
- BeanDefinitionReader
- AnnotationConfigApplicationContext
- Programmatic bean registration
- Dynamic bean registration
- BeanDefinitionRegistryPostProcessor

## 13. ApplicationContext
- ApplicationContext
- ApplicationContext hierarchy
- Parent and child contexts
- Context initialization
- Context refresh
- Context shutdown
- refresh()
- MessageSource
- ApplicationEventPublisher
- ResourceLoader
- EnvironmentCapable
- ApplicationContextAware

## 14. Circular Dependencies
- Circular dependency
- Constructor circular dependency
- Setter circular dependency
- Field circular dependency
- Early singleton exposure
- Three-level singleton cache
- singletonObjects
- earlySingletonObjects
- singletonFactories
- Circular dependency resolution
- Why constructor circular dependency fails
- Redesigning circular dependencies
- @Lazy
- Setter injection
- ObjectProvider

## 15. Lazy Initialization
- Eager initialization
- Lazy initialization
- @Lazy
- Lazy bean creation
- Lazy dependency
- Application startup impact
- Lazy initialization trade-offs

## 16. Profiles
- Spring Profiles
- @Profile
- Active profiles
- Default profile
- Profile-specific configuration
- Profile expressions
- Environment profiles

## 17. Environment and Properties
- Environment abstraction
- PropertySource
- MutablePropertySources
- Property resolution
- @Value
- Property placeholders
- ${...}
- application properties
- Environment variables
- System properties
- External configuration
- Property precedence

## 18. Spring Expression Language (SpEL)
- SpEL fundamentals
- Literal expressions
- Property access
- Method invocation
- Operators
- Conditional expressions
- Collection selection
- Collection projection
- Bean references
- @Value with SpEL

## 19. Spring Events
- Application events
- ApplicationEvent
- ApplicationEventPublisher
- ApplicationListener
- @EventListener
- Custom events
- Synchronous events
- Asynchronous events
- @Async with events
- Event ordering
- Transaction-bound events
- @TransactionalEventListener

## 20. Resource Handling
- Resource abstraction
- ResourceLoader
- ResourcePatternResolver
- ClassPathResource
- FileSystemResource
- UrlResource
- InputStreamResource
- ByteArrayResource
- Resource patterns
- classpath resources
- File resources

## 21. Internationalization
- MessageSource
- ResourceBundleMessageSource
- ReloadableResourceBundleMessageSource
- Message resolution
- Locale
- LocaleResolver
- LocaleContext
- Message arguments
- messages.properties
- Locale-specific messages

## 22. Type Conversion
- ConversionService
- Converter
- GenericConverter
- ConverterFactory
- Formatter
- FormatterRegistry
- Default conversion service
- Custom converters
- TypeDescriptor

## 23. Validation
- Spring Validator
- Validator interface
- Errors
- BindingResult
- DataBinder
- ValidationUtils
- JSR-303 / Jakarta Bean Validation integration
- Custom validators

## 24. Data Binding
- DataBinder
- WebDataBinder
- Property binding
- Property editors
- Conversion service
- Binding errors
- Custom property editors

## 25. Spring AOP
- Aspect-Oriented Programming
- Cross-cutting concerns
- Aspect
- Join point
- Pointcut
- Advice
- Target object
- Proxy
- Introduction
- Weaving
- @Aspect
- @Before
- @After
- @AfterReturning
- @AfterThrowing
- @Around
- Pointcut expressions

## 26. Spring Proxy Mechanism
- Proxy-based AOP
- JDK dynamic proxy
- CGLIB proxy
- JDK proxy vs CGLIB
- ProxyTargetClass
- TargetSource
- Advised
- AopProxy
- ProxyFactory
- AspectJProxyFactory
- Self-invocation problem
- Exposing proxy
- AopContext

## 27. Transaction Management
- Spring transaction abstraction
- PlatformTransactionManager
- TransactionDefinition
- TransactionStatus
- TransactionTemplate
- Declarative transactions
- Programmatic transactions
- @Transactional
- Transaction proxy
- Transaction interception
- Transaction propagation
- REQUIRED
- REQUIRES_NEW
- SUPPORTS
- NOT_SUPPORTED
- MANDATORY
- NEVER
- NESTED
- Transaction isolation
- READ_UNCOMMITTED
- READ_COMMITTED
- REPEATABLE_READ
- SERIALIZABLE
- Read-only transactions
- Timeout
- Rollback rules
- Checked vs unchecked exceptions
- Self-invocation and transactions

## 28. Task Execution and Scheduling
- TaskExecutor
- ThreadPoolTaskExecutor
- TaskScheduler
- ThreadPoolTaskScheduler
- @Async
- @EnableAsync
- @Scheduled
- @EnableScheduling
- Async exception handling
- Executor configuration
- Thread pool configuration

## 29. Spring Resource and Application Infrastructure
- ResourceLoader
- ApplicationEventPublisher
- MessageSource
- Environment
- ApplicationContextAware
- ResourcePatternResolver
- ApplicationStartup
- Startup metrics

## 30. Spring Context Startup Internals
- ApplicationContext creation
- Context refresh lifecycle
- BeanFactory creation
- BeanDefinition loading
- BeanDefinition post-processing
- BeanFactoryPostProcessor execution
- BeanPostProcessor registration
- Singleton pre-instantiation
- Event infrastructure initialization
- ApplicationContext publication
- Context shutdown

## 31. Spring Internals
- DefaultListableBeanFactory
- AbstractApplicationContext
- AbstractAutowireCapableBeanFactory
- AbstractBeanFactory
- SingletonBeanRegistry
- DefaultSingletonBeanRegistry
- BeanDefinitionRegistry
- Dependency resolution internals
- Bean creation internals
- Bean initialization internals
- Bean destruction internals
- Reflection in Spring
- CGLIB usage
- Dynamic proxy usage

## 32. Singleton Internals
- Container-managed singleton
- Singleton registry
- Singleton cache
- singletonObjects
- earlySingletonObjects
- singletonFactories
- Singleton creation process
- Early singleton exposure
- Thread safety considerations

## 33. FactoryBean
- FactoryBean interface
- getObject()
- getObjectType()
- isSingleton()
- FactoryBean vs Bean
- &beanName lookup
- Common FactoryBean use cases

## 34. Ordering
- @Order
- Ordered
- PriorityOrdered
- Ordering BeanPostProcessors
- Ordering event listeners
- Ordering injected collections
- Ordering interceptors

## 35. Conditional Bean Registration
- @Conditional
- Condition interface
- ConditionContext
- AnnotatedTypeMetadata
- Custom conditions
- Conditional configuration concepts

## 36. Spring Design Patterns
- Singleton Pattern
- Factory Pattern
- Factory Method Pattern
- Proxy Pattern
- Template Method Pattern
- Strategy Pattern
- Observer Pattern
- Adapter Pattern
- Decorator Pattern
- Front Controller Pattern

## 37. Spring Boot Core Integration
- SpringApplication
- @SpringBootApplication
- Auto-configuration
- Component scanning
- Configuration
- Spring Boot starters
- Conditional auto-configuration
- @EnableAutoConfiguration
- @ConditionalOnClass
- @ConditionalOnMissingBean
- @ConditionalOnBean
- @ConditionalOnProperty
- Auto-configuration ordering
- Auto-configuration debugging
- Application startup lifecycle

## 38. Testing with Spring
- Spring TestContext Framework
- @SpringBootTest
- @ContextConfiguration
- @ActiveProfiles
- @TestConfiguration
- @MockBean
- @SpyBean
- ApplicationContext testing
- Integration testing
- Test slices
- Context caching
- Test lifecycle

## 39. Production and Performance Considerations
- Application startup time
- Bean creation overhead
- Lazy initialization
- Component scanning overhead
- Excessive bean creation
- Singleton memory usage
- Proxy overhead
- Thread safety of singleton beans
- Context hierarchy
- Configuration optimization
- Startup troubleshooting
- Bean creation failures
- Dependency resolution failures

## 40. Troubleshooting and Debugging
- NoSuchBeanDefinitionException
- NoUniqueBeanDefinitionException
- BeanCreationException
- BeanCurrentlyInCreationException
- UnsatisfiedDependencyException
- ApplicationContext startup failures
- Circular dependency debugging
- Autowiring failures
- Configuration conflicts
- Profile-related issues
- Proxy-related issues
- Transaction proxy issues
- Startup condition debugging

## 41. Advanced Spring Core Topics
- ApplicationContext hierarchy
- Parent-child containers
- Custom scopes
- Custom BeanPostProcessor
- Custom BeanFactoryPostProcessor
- Custom BeanDefinitionRegistryPostProcessor
- Custom ApplicationContext
- Custom ApplicationEvent
- Custom ApplicationListener
- Custom Condition
- Custom Converter
- Custom Formatter
- Custom Validator
- Custom Scope
- Programmatic bean registration
- Dynamic bean creation
- ApplicationContext events
- Application startup instrumentation
