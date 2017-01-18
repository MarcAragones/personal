# Mockito

## Mockito Verify Example

From: [Java Code Geeks](https://examples.javacodegeeks.com/core-java/mockito/mockito-verify-example/)

### 1. System Under Test (SUT)

```java
package com.javacodegeeks.mockito;

public class Customer {
	private AccountManager accountManager;

	public long withdraw(long amount) throws NotEnoughFundsException {
		Account account = accountManager.findAccount(this);
		long balance = accountManager.getBalance(account);
		if (balance < amount) {
			throw new NotEnoughFundsException();
		}
		accountManager.withdraw(account, amount);
		return accountManager.getBalance(account);
	}

	public void setAccountManager(AccountManager accountManager) {
		this.accountManager = accountManager;
	}
}
```

### 2. Verify Behaviour

We will try to withdraw more amount than is allowed.

```java
public class MockitoVerifyExample {
	private Customer customer;
	private AccountManager accountManager;
	private Account account;
	private long withdrawlAmount2000 = 2000L;
	
	@BeforeMethod
	public void setupMock() {
		customer = new Customer();
		accountManager = mock(AccountManager.class);
		customer.setAccountManager(accountManager);
		account = mock(Account.class);
		when(accountManager.findAccount(customer)).thenReturn(account);		
	}
	
	@Test(expectedExceptions=NotEnoughFundsException.class)
	public void withdrawButNotEnoughFunds() throws NotEnoughFundsException {
		long balanceAmount200 = 200L;
		
		// Stub AccountManager to return balance lesser than the amount requested.
		p("Train getBalance(account) to return " + balanceAmount200);
		when(accountManager.getBalance(account)).thenReturn(balanceAmount200);
		
		printBalance(balanceAmount200);
		try {
			p("Customer.withdraw(" + withdrawlAmount2000 + ") should fail with NotEnoughFundsException");
			
			// Run the SUT method
			customer.withdraw(withdrawlAmount2000);			
		} catch (NotEnoughFundsException e) {
			p("NotEnoughFundsException is thrown"); 
			
			verify(accountManager).findAccount(customer);
			p("Verified findAccount(customer) is called");			
			
			verify(accountManager, times(0)).withdraw(account, withdrawlAmount2000);
			p("Verified withdraw(account, " + withdrawlAmount2000 + ") is not called");
			
			throw e;
		}
	}

	private static void p(String text) {
		System.out.println(text);
	}
	
	private void printBalance(long balance) {
		p("Balance is " + balance + " and withdrawl amount " + withdrawlAmount2000);	
	}
}
```

Output:

```java
Train getBalance(account) to return 200
Balance is 200 and withdrawl amount 2000
Customer.withdraw(2000) should fail with NotEnoughFundsException
NotEnoughFundsException is thrown
Verified findAccount(customer) is called
Verified withdraw(account, 2000) is not called
PASSED: withdrawButNotEnoughFunds
```

## @Mock, @Spy, @Captor and @InjectMocks

From: [Baeldung](http://www.baeldung.com/mockito-annotations)

### 1. Enable Mockito Annotations

Annotate the JUnit test with a runner – `MockitoJUnitRunner`:

```java
@RunWith(MockitoJUnitRunner.class)
public class MockitoAnnotationTest {
    // ...
}
```

Programmatically alternative:

```java
@Before
public void init() {
    MockitoAnnotations.initMocks(this);
}
```

### 2. @Mock Annotation

We can use @Mock to create and inject mocked instances without having to call Mockito.mock manually.

This:

```java
@Test
public void whenNotUseMockAnnotation_thenCorrect() {
    List mockList = Mockito.mock(ArrayList.class);
     
    mockList.add("one");
    Mockito.verify(mockList).add("one");
    assertEquals(0, mockList.size());
 
    Mockito.when(mockList.size()).thenReturn(100);
    assertEquals(100, mockList.size());
}
```

is equivalent to this:

```java
@Mock
List<String> mockedList;
 
@Test
public void whenUseMockAnnotation_thenMockIsInjected() {
    mockedList.add("one");
    Mockito.verify(mockedList).add("one");
    assertEquals(0, mockedList.size());
 
    Mockito.when(mockedList.size()).thenReturn(100);
    assertEquals(100, mockedList.size());
}
```

### 3. @Spy Annotation

With a *mock*, `mockList.add("one")` didn't really add an element. Spy uses the **real** method of an instance.

Like with @Mock, this:

```java
List<String> spyList = Mockito.spy(new ArrayList<String>());
```

is equivalent to this:

```java
@Spy
List<String> spiedList = new ArrayList<String>();
```

Example:

```java
@Test
public void whenNotUseSpyAnnotation_thenCorrect() {
    List<String> spyList = Mockito.spy(new ArrayList<String>());
     
    spyList.add("one");
    spyList.add("two");
 
    Mockito.verify(spyList).add("one");
    Mockito.verify(spyList).add("two");
 
    assertEquals(2, spyList.size());
 
    Mockito.doReturn(100).when(spyList).size();
    assertEquals(100, spyList.size());
}
```

### 4. @Caption Annotation

@Caption creates an `ArgumentCaptor` instance.

This:

```java
List mockList = Mockito.mock(List.class);
ArgumentCaptor<String> arg = ArgumentCaptor.forClass(String.class);
```

is equivalent to this:

```java
@Mock
List mockedList;
 
@Captor
ArgumentCaptor argCaptor;
```

Example:

```java
@Mock
List mockedList;
 
@Captor
ArgumentCaptor argCaptor;
 
@Test
public void whenUseCaptorAnnotation_thenTheSam() {
    mockedList.add("one");
    Mockito.verify(mockedList).add(argCaptor.capture());
 
    assertEquals("one", argCaptor.getValue());
}
```

### 5. @InjectMocks Annotation

@InjectMocks annotation injects mock fields into the tested object automatically.

```java
@Mock
Map<String, String> wordMap;
 
@InjectMocks
MyDictionary dic = new MyDictionary();
 
@Test
public void whenUseInjectMocksAnnotation_thenCorrect() {
    Mockito.when(wordMap.get("aWord")).thenReturn("aMeaning");
 
    assertEquals("aMeaning", dic.getMeaning("aWord"));
}
```

And here is the class `MyDictionary`:

```java
public class MyDictionary {
    Map<String, String> wordMap;
 
    public MyDictionary() {
        wordMap = new HashMap<String, String>();
    }
    public void add(final String word, final String meaning) {
        wordMap.put(word, meaning);
    }
    public String getMeaning(final String word) {
        return wordMap.get(word);
    }
}
```