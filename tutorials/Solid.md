
# SOLID

From: [Tuts+](https://code.tutsplus.com/tutorials/solid-part-1-the-single-responsibility-principle--net-36074)

## Single Responsibility Principle

### The Definition

> A class should have only one reason to change.

### The Audience

Audience defines reasons for change. Here are a couple of modules and their possible audiences:

- **Persistence Module** - Audience include DBAs and software architects.
- **Reporting Module** - Audience include clerks, accountants, and operations.
- **Payment Computation Module for a Payroll System** - Audience may include lawyers, managers, and accountants.
- **Book Search Module for a Library Management System** - Audience may include the librarian and/or the clients themselves.

### Actors and Roles

The actors define the audience. This greatly helps us to reduce the concept of concrete persons like "John the architect" to Architecture, or "Mary the referent" to Operations.

> So a responsibility is a family of functions that serves one particular actor. (Robert C. Martin)

### Source of Change

> An actor for a responsibility is the single source of change for that responsibility. (Robert C. Martin)

### Classic Examples

#### Objects That Can "Print" Themselves

```php
class Book {
 
    function getTitle() {
        return "A Great Book";
    }
 
    function getAuthor() {
        return "John Doe";
    }
 
    function turnPage() {
        // pointer to next page
    }
 
    function printCurrentPage() {
        echo "current page content";
    }
}
```

There are **2 different actors** here: Book Management (like the librarian) and Data Presentation Mechanism (like the way we want to deliver the content to the user - on-screen, graphical UI, text-only UI, maybe printing).

Mixing business logic with presentation is bad because it is against the Single Responsibility Principle (SRP).

Separating presentation from business logic, and respecting SRP, gives great advantages in our design's flexibility:

```php
class Book {
 
    function getTitle() {
        return "A Great Book";
    }
 
    function getAuthor() {
        return "John Doe";
    }
 
    function turnPage() {
        // pointer to next page
    }
 
    function getCurrentPage() {
        return "current page content";
    }
 
}

interface Printer {
 
    function printPage($page);
}
 
class PlainTextPrinter implements Printer {
 
    function printPage($page) {
        echo $page;
    }
 
}
 
class HtmlPrinter implements Printer {
 
    function printPage($page) {
        echo '<div style="single-page">' . $page . '</div>';
    }
 
}
```

#### Objects That Can "Save" Themselves

```php
class Book {
 
    function getTitle() {
        return "A Great Book";
    }
 
    function getAuthor() {
        return "John Doe";
    }
 
    function turnPage() {
        // pointer to next page
    }
 
    function getCurrentPage() {
        return "current page content";
    }
 
    function save() {
        $filename = '/documents/'. $this->getTitle(). ' - ' . $this->getAuthor();
        file_put_contents($filename, serialize($this));
    }
 
}
```

We can, again identify several actors like Book Management System and Persistence. Whenever we want to change persistence, we need to change this class. Whenever we want to change how we get from one page to the next, we have to modify this class. There are several axis of change here.

```php
class Book {
 
    function getTitle() {
        return "A Great Book";
    }
 
    function getAuthor() {
        return "John Doe";
    }
 
    function turnPage() {
        // pointer to next page
    }
 
    function getCurrentPage() {
        return "current page content";
    }
 
}
 
class SimpleFilePersistence {
 
    function save(Book $book) {
        $filename = '/documents/' . $book->getTitle() . ' - ' . $book->getAuthor();
        file_put_contents($filename, serialize($book));
    }
 
}
```

Moving the persistence operation to another class will clearly separate the responsibilities and we will be free to exchange persistence methods without affecting our `Book` class. For example implementing a `DatabasePersistence` class would be trivial and our business logic built around operations with books will not change.

### A Higher Level View

![High level view](https://cdn.tutsplus.com/net/uploads/2013/12/HighLevelDesign.png)

**The Single Responsibility Principle is respected**. Object creation is separated on the right in Factories and the main entry point of our application, one actor one responsibility. Persistence is also taken care of at the bottom. A separate module for the separate responsibility. Finally, on the left, we have presentation or the delivery mechanism if you wish, in the form of an MVC or any other type of UI. SRP respected again. All that remains is to figure out what to do inside of our business logic.

### Software Design Considerations

The primary value of software is **ease of change**. The secondary is functionality, in the sense of satisfying as much requirements as possible, meeting the user's needs. However, in order to achieve a **high secondary value, a primary value is mandatory**. To keep our primary value high, we must have a design that is easy to change, to extend, to accommodate new functionalities and to ensure that SRP is respected.

We can reason in a step by step manner:

1. High primary value leads in time to high secondary value.
2. Secondary value means needs of the users.
3. Needs of the users means needs of the actors.
4. Needs of the actors determines the needs of changes of these actors.
5. Needs of change of actors defines our responsibilities.

So when we design our software we should:

1. Find and define the actors.
2. Identify the responsibilities that serve those actors.
3. Group our functions and classes so that each has only one allocated responsibility.

### A Less Obvious Example

```php
class Book {
 
    function getTitle() {
        return "A Great Book";
    }
 
    function getAuthor() {
        return "John Doe";
    }
 
    function turnPage() {
        // pointer to next page
    }
 
    function getCurrentPage() {
        return "current page content";
    }
 
    function getLocation() {
        // returns the position in the library
        // ie. shelf number & room number
    }
 
}
```

The function `getLocation()` may be the problem.

All of the methods of the Book class are about business logic. So our perspective must be from the business's point of view. If our application is written to be used by real librarians who are searching for books and giving us a physical book, then **SRP might be violated**.

We can reason that the actor operations are the ones interested in the methods `getTitle()`, `getAuthor()` and `getLocation()`. The clients may also have access to the application to select a book and read the first few pages to get an idea about the book and decide if they want it or not. So the actor readers may be interested in all the methods except `getLocations()`. **An ordinary client doesn't care where the book is kept in the library**. The book will be handed over to the client by the librarian. So, we do indeed have a violation of SRP.

```php
class Book {
 
    function getTitle() {
        return "A Great Book";
    }
 
    function getAuthor() {
        return "John Doe";
    }
 
    function turnPage() {
        // pointer to next page
    }
 
    function getCurrentPage() {
        return "current page content";
    }
 
}
 
class BookLocator {
 
    function locate(Book $book) {
        // returns the position in the library
        // ie. shelf number & room number
        $libraryMap->findBookBy($book->getTitle(), $book->getAuthor());
    }
 
}
```

Introducing the BookLocator, the librarian will be interested in the BookLocator. The client will be interested in the Book only. Of course, there are several ways to implement a BookLocator. It can use the author and title or a book object and get the required information from the Book. It always depends on our business. What is important is that if the library is changed, and the librarian will have to find books in a differently organized library, the Book object will not be affected. In the same way, if we decide to provide a pre-compiled summary to the readers instead of letting them browse the pages, that will not affect the librarian nor the process of finding the shelf the books sits on.

### CodelyTV Example

From: [CodelyTV](https://www.youtube.com/watch?v=c97P1UmF1cs)

```php
class UserManager {

	private $database_manager;
	private $password_manager;

	public function __constructor(DatabaseManager $database_manager, PasswordManager $password_manager) {
		$this->database_manager = $database_manager;
		$this->password_manager = $password_manager;
	}

	public function checkRegisteredUser($email, $password) {
		// Check if user exists
	}

	public function registerNewUser($email, $password) {
		// Create a new user
	}
}
```

What if we want to send an email when registering a new user? New constructor:

```php
public function __constructor(DatabaseManager $database_manager, PasswordManager $password_manager, $email_sender) {
		$this->database_manager = $database_manager;
		$this->password_manager = $password_manager;
		$this->email_sender = $email_sender;
	}
```

**Problem:** We are injecting a dependency for the method `checkRegisteredUser` that it doesn't need!

**Solution:**

```php
class RegisterNewUserUseCase {

	private $database_manager;
	private $password_manager;

	public function __constructor(DatabaseManager $database_manager, PasswordManager $password_manager) {
		$this->database_manager = $database_manager;
		$this->password_manager = $password_manager;
	}

	public function invoke($email, $password) {
		// Create a new user
	}
}

class CheckRegisteredUserUseCase {

	private $database_manager;
	private $password_manager;
	private $email_sender;

	public function __constructor(DatabaseManager $database_manager, PasswordManager $password_manager, EmailSender $email_sender) {
		$this->database_manager = $database_manager;
		$this->password_manager = $password_manager;
		$this->email_sender = $email_sender;
	}

	public function invoke($email, $password) {
		// Check if user exists
	}
}
```

## Open/Close Principle

### The Definition

> Software entities (classes, modules, functions, etc.) should be open for extension, but closed for modification.

We should **not modify our existing code** but rather write new code that will be used by existing code. We want to create new features in ways that will not require us to redeploy existing binaries, executables or DLLs.

### OCP in the SOLID Context

If we think about OCP and SRP, we can observe that they are complementary. When we have code that has a single reason to change, introducing a new feature will create a secondary reason for that change. So both SRP and OCP would be violated.

### The Obvious Example of OCP Violation

This violates the OCP:

![OCP violation](https://cdn.tutsplus.com/net/uploads/2014/01/violate1.png)

If we need to **implement a second `Logic`** class in a way that will allow us to use both the current one and the new one, the existing `Logic` class will need to be changed.

There is **no way for us to provide a new `Logic` without affecting the current one.**

When we are talking about statically typed languages, it is very possible that the **`User` class will also require changes**.

If we are talking about compiled languages, most certainly both the `User` executable and the `Logic` executable or dynamic library will **require recompilation and redeployment** to our clients.

### Example

Let's say we want to write a class that can provide progress as a percent for a file that is downloaded through our application. We will have two main classes, a `Progress` and a `File`, and I imagine we will want to use them like in the test below.

```php
function testItCanGetTheProgressOfAFileAsAPercent() {
    $file = new File();
    $file->length = 200;
    $file->sent = 100;
 
    $progress = new Progress($file);
 
    $this->assertEquals(50, $progress->getAsPercent());
}

class File {
    public $length;
    public $sent;
}

class Progress {
 
    private $file;
 
    function __construct(File $file) {
        $this->file = $file;
    }
 
    function getAsPercent() {
        return $this->file->sent * 100 / $this->file->length;
    }
 
}
```

This code seems to be right, however it violates the Open/Closed Principle. But why? And How?

### Changing Requirements

New feature: adding `Music` that the percentage is based on seconds not bytes. This makes we can't reuse `Progress`.

In order to do that we have to modify it, we have to make `Progress` know about `Music` and `File`.

If our design would respect OCP, we would not need to touch `File` or `Progress`.

### Solution: Use the Strategy Design Pattern

![Strategy Design Pattern](https://cdn.tutsplus.com/net/uploads/2014/01/strategy.png)

With `Measurable` interface, we only have to add a new class if we want a new strategy. And we don't have to change `Progress` or previous strategies.

```php
interface Measurable {
    function getLength();
    function getSent();
}

class Progress {
 
    private $measurableContent;
 
    function __construct(Measurable $measurableContent) {
        $this->measurableContent = $measurableContent;
    }
 
    function getAsPercent() {
        return $this->measurableContent->getSent() * 100 / $this->measurableContent->getLength();
    }
 
}
```