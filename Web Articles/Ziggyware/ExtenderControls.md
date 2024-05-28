# Extender Controls


Ever wondered how that tool tip control adds properties to all of your controls on a windows form? We can achieve this with a user control by implementing the IExtenderProvider interface. You also need to specify a few attributes to the class. Consider the following:

```csharp
    [Category("Validation")]
    [Description("Type of validating performend on this control.")]
    [ProvideProperty("ValidationType", typeof(Control))]
    [ProvideProperty("ValidationText", typeof(Control))]
    [ProvideProperty("Required", typeof(Control))]
    public partial class HLInputValidator : Component, IExtenderProvider
```

The class "HLInputValidator" is a user control I made that allows all the controls on a form to go through a detailed validation system before executing its purpose. It extends text boxes with a validation property with properties like Date, PositiveInteger, NegativeDouble, WindowsUsername, and any other validation types I wish to imply on my controls.

You implement the "CanExtend" method which is called when the designer sends the request to your extender control for whether or not a form control on the form can be extended. In the following example, any control can be extended (type Control) and always remember to prevent adding yourself.

```csharp
        bool IExtenderProvider.CanExtend(object target)
        {
            if (target is Control && !(target is HLInputValidator))
                return true;

            return false;
        }
```

You then need to implement a Get/Set pair of methods for each of your extended properties. Example:

```csharp
        [DefaultValue(false)]
        public bool GetRequired(Control control)
        {
            if (!requiredControls.Contains(control))
                return false;

            bool req = (bool)requiredControls[control];
            return req;
        }

        public void SetRequired(Control control, bool value)
        {
            if (value == false)
            {
                requiredControls.Remove(control);
            }
            else
            {
                requiredControls[control] = value;
            }
        }
```

The "requiredControls" is an ArrayList used to store the form controls in the extender user control.

Try to get your head around this handy interface as it can be extremely powerful for how you can use it (as shown, it makes for some powerful validation systems).
