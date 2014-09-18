@DeclareRoles, @RolesAllowed

```java
@DeclareRoles("specified_in_declare_roles")
@Stateless
public class SecuredBean {

	private static final Logger logger = Logger.getLogger(SecuredBean.class.getName());

	@Resource
	private EJBContext ejbContext;
```
```java
	@RolesAllowed("not_specified_in_declare_roles")
	public void a() {
		logger.log(Level.INFO, "called by {0}", ejbContext.getCallerPrincipal().getName());
	}
```
- Role nélkül: `javax.ejb.AccessLocalException: Client not authorized for this invocation`

```java
	@RolesAllowed("specified_in_declare_roles")
	public void b() {
		logger.log(Level.INFO, "called by {0}", ejbContext.getCallerPrincipal().getName());
	}
```
- Role nélkül: `Could not call method "b"`

```java
	public void c() {
		final boolean allowed = ejbContext.isCallerInRole("not_specified_in_declare_roles");
		logger.log(Level.INFO, "called by {0}, allowed: {1}", new Object[]{ejbContext.getCallerPrincipal().getName(), allowed});
	}
```
- Role nélkül: `called by ANONYMOUS, allowed: false`

```java
	public void d() {
		final boolean allowed = ejbContext.isCallerInRole("specified_in_declare_roles");
		logger.log(Level.INFO, "called by {0}, allowed: {1}", new Object[]{ejbContext.getCallerPrincipal().getName(), allowed});
	}
}
```
- Role nélkül: `called by ANONYMOUS, allowed: false`


@RunAs("not_specified_in_declare_roles") - java.lang.RuntimeException: The RunAs role "not_specified_in_declare_roles" is not mapped to a principal.
@RunAs("specified_in_declare_roles") - java.lang.RuntimeException: The RunAs role "specified_in_declare_roles" is not mapped to a principal.



A @RolesAllowed annotation implicitly declares a role that will be referenced in the application; therefore, no @DeclareRoles annotation is required. The presence of the @RolesAllowed annotation also implicitly declares that authentication will be required for a user to access these methods. If no authentication method is specified in the deployment descriptor, the type of authentication will be user name/password authentication.
