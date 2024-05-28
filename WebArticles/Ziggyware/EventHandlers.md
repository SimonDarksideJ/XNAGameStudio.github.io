# Event Handlers


Here is a small example on adding your own events to the events list of a user control.

Event handlers in C# are known as "Delegates". We declare a delegate outside of our class like so:

```csharp
public delegate void MyEventHandler(object sender);
```

Now here is the class declaration of a simple user control: 

```csharp
public partial class UserControl1 : UserControl
    {
public UserControl1()
        {
            InitializeComponent();
        }

        [Description("The custom event.")]
        public event MyEventHandler MyEvent;
    }
```

To call this event from somewhere in the user control, you simply do: 

```csharp
    if (MyEvent != null)
        MyEvent(this);
```

Now when you instance this control on your forms, you will see an event called "MyEvent" in the events list. Double click on that to insert a handler for that event in your forms code. 