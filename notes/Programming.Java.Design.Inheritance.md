---
id: wlw06wkxhmk1f68x0hvzcw5
title: Inheritance
desc: ''
updated: 1669374558970
created: 1669374152312
---
Inheritance is for code reuse and the DRY (Don't Repeat Yourself) principle.

### Composition
To implement a **has-a** relationship, using an *instance variable* that refers to other objects, if an object contains the other object and the contained object can't exist without the existence of the original object then it is **composition**.

*Composition is a way of describing reference between two or more classes using instance variable and an instance should be created before use*.

Pros:
- Allows code reuse
- Java doesn't support multiple inheritance but by using composition we can achieve it
- Composition offers better testability of a class
- Through using it, we are flexible enough to replace implementation of a composed class with better and improved version
- By using composition, we can also change the member objects at run time, to dynamically change program behaviour

[[Example | https://www.geeksforgeeks.org/composition-in-java/]]

```Java
// Java program to Illustrate Concept of Composition
 
// Importing required classes
import java.io.*;
import java.util.*;
 
// Class 1
// Helper class
// Book class
class Book {
 
    // Member variables of this class
    public String title;
    public String author;
 
    // Constructor of this class
    Book(String title, String author)
    {
 
        // This keyword refers top current instance
        this.title = title;
        this.author = author;
    }
}
 
// Class 2
// Helper class
// Library class contains list of books.
class Library {
 
    // Reference to refer to list of books.
    private final List<Book> books;
 
    // Constructor of this class
    Library(List<Book> books)
    {
 
        // This keyword refers to current instance itself
        this.books = books;
    }
 
    // Method of this class
    // Getting the list of books
    public List<Book> getListOfBooksInLibrary()
    {
        return books;
    }
}
 
// Class 3
// Main class
class GFG {
 
    // Main driver method
    public static void main(String[] args)
    {
 
        // Creating the objects of class 1 (Book class)
        // inside main() method
        Book b1
            = new Book("EffectiveJ Java", "Joshua Bloch");
        Book b2
            = new Book("Thinking in Java", "Bruce Eckel");
        Book b3 = new Book("Java: The Complete Reference",
                           "Herbert Schildt");
 
        // Creating the list which contains the
        // no. of books.
        List<Book> book = new ArrayList<Book>();
 
        // Adding books to List object
        // using standard add() method
        book.add(b1);
        book.add(b2);
        book.add(b3);
 
        // Creating an object of class 2
        Library library = new Library(book);
 
        // Calling method of class 2 and storing list of
        // books in List Here List is declared of type
        // Books(user-defined)
        List<Book> books
            = library.getListOfBooksInLibrary();
 
        // Iterating over for each loop
        for (Book bk : books) {
 
            // Print and display the title and author of
            // books inside List object
            System.out.println("Title : " + bk.title
                               + " and "
                               + " Author : " + bk.author);
        }
    }
}
```