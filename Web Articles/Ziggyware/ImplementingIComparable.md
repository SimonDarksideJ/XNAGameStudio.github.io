# Implementing IComparable


IComparable is used when sorting elements in a HashTable or ArrayList 

```csharp
using System;
using System.Collections;
```


Lets start off by declaring a namespace and class for our test application: 

```csharp
namespace testApp
{
    public class TestApp
    {
```

Create a class that derives from the interface IComparable: 

```csharp
        class Person : IComparable
        {
            public string Name;
            public string ID;

            public Person(string Name,string ID)
            {
                this.Name = Name;
                this.ID = ID;
            }
```


Implement the System.IComparable.CompareTo function to perform a custom sort of the elements: 

```csharp
            int System.IComparable.CompareTo(object obj)
            {
                Person p = obj as Person;

                return string.Compare(Name,p.Name);
            }
```


override the ToString() function to display a custom output 

```csharp
            public override string ToString()
            {
                return "ID: " + ID + " Name: " + Name;
            }
            
        }
```


Now lets declare the main entry point into our application: 

```csharp
        static void Main() 
        {
```            


We can test our new class's IComparable.CompareTo() method by populating an ArrayList and invoking the Sort() method: 

```csharp
            ArrayList ar = new ArrayList();
            
            ar.Add(new Person("Bob","123"));
            ar.Add(new Person("Al", "623"));
            ar.Add(new Person("Jim","023"));

            ar.Add(new Person("Joe","611"));
            ar.Add(new Person("Robert", "923"));
            ar.Add(new Person("Flint","013"));

            ar.Sort();
```


Lets see if the data was sorted as we expected: 

```csharp
            foreach(object o in ar)
                Console.WriteLine(o.ToString());
        }
    }
}
```

The output of this should be sorted by the Name field of the class that was derived from IComparable: 

```
ID: 623 Name: Al
ID: 123 Name: Bob
ID: 013 Name: Flint
ID: 023 Name: Jim
ID: 611 Name: Joe
ID: 923 Name: Robert
```

Looks like it worked flawlessly!
