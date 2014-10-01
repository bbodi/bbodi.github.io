---
layout: default
title: Private field injection into parent classes!
---

Munka közben felmerült egy érdekes kérdés, amire nem találtam azonnal egyértelmű választ az EJB specifikációban (és a fejemben).


Legyen `Parent` egy sima osztály, és `Child` `Parent` egy gyermekosztálya. `Parent`-en nincs semmilyen annotáció, viszont `Child` egy Session Bean.
- Működik-e a field injection a Parent osztályban `protected` field esetén?
- Működik-e a field injection a Parent osztályban `private` field esetén? Míg a protected verzióra a válasz magától értetődőnek tűnik, addig a `private` már egyáltalán nem triviális.
- Ha működnek, miért? A specifikáció mely részei utalnak a problémára?

###A teszt

A `Child` osztály egy `@Singleton` lesz, amely szülőjébe beinjektálok egy producer által előállított számot. A `Child` egyszerűen kiírja az előállított számot.

```java
public class Parent {
	@Inject
	private int privateField;

	@Inject
	protected int protectedField;

	public int getPrivateField() {
		return privateField;
	}

	public int getProtectedField() {
		return protectedField;
	}
}
```
Ez lesz a `Parent` osztályunk. Egy sima osztály egy `private` és egy `protected` injektált fielddel.
Az injektáláshoz a következő osztályt használjuk, ami egy `Producer` metódust ajánl ki `int` típushoz.
```java
@Singleton
public class SomeNumberProducer {
	private int counter = 3;

	@Produces
	public Integer someNumber() {
		return counter++;
	}
}
```
Biztos akarok lenni benne, hogy a két injektált field két külön injektálási folyamatot jelent, ezért használok egy `@Singleton` bean-t, ami lehetővé teszi, hogy minden injektálásnál növeljem a beinjektált értéket (`counter`). A `counter` azért kapott default 3-as értéket, mert a 0-ról nem lehetne eldönteni, hogy az az injektálás miatt lett 0, vagy egyszerűen az integer default értékére lett beállítva a változó.

Most pedig következzen a gyermek osztály:
```java
@Singleton
@Startup
public class Child extends Parent {
	private static final Logger logger = Logger.getLogger(Child.class.getName());

	@PostConstruct
	private void init() {
		logger.log(Level.INFO, "value of private field: {0}", this.getPrivateField());
		logger.log(Level.INFO, "value of protected field: {0}", this.getProtectedField());
	}
}
```
A Child egy `Singleton bean`, aminek az `init()` metóedusa az alkalmazás-szerver indulásakor lefut, hála a `@Startup`, `@PostConstruct` párosításnak.

Az eredmény az elvárásaimnak megfelelő: mindkét field-be sikeres volt az injektálás:
```
value of private field: 3
value of protected field: 4
```

### EJB specifikáció
Végül következzen a megfelelő rész a specifikációból, ami feketén-fehéren megmagyarázza a fentieket:

#### 4.9.2.1 Session Bean Superclasses

> A session bean class is permitted to have superclasses that are themselves session bean classes. However, there are no special rules that apply to the processing of annotations or the deployment descriptor
for this case.
For the purposes of processing a particular session bean class, **all superclass processing is identical regardless of whether the superclasses are themselves session bean classes**. In this regard, the
use of session bean classes as superclasses merely represents a convenient use of implementation inheritance, but does not have component inheritance semantics.

#### 4.9.2. Session Bean Class
Nem tartozik szorosan a blogbejegyzéshez, de a alábbi részlet is érdekes lehet:
> The session bean class may have superclasses and/or superinterfaces. If the session bean has superclasses,
- the business methods,
- lifecycle callback interceptor methods,
- the timeout callback methods,
- the methods implementing the optional session synchronization notifications,
- the `Init` or `ejbCreate<Method>` methods,
- the `Remove` methods,
- and the methods of the `SessionBean` interface,
may be defined in the session bean class, or in any of its superclasses.
