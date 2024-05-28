# Using the WebBrowser COM Object


The WebBrowser COM object is a valuable tool when you need rich dynamic content in your application or would just like to display web based content.

In .NET 1.1 you can create one of these objects by adding it to the tool panel:

Right click on the tool panel and choose Add/Remove Items
Now select COM Components and look for the "Microsoft Web Browser"

```csharp
private AxSHDocVw.AxWebBrowser webBrowser;
```

I created a web control on my form called webBrowser

Lets ad some content to the form. First we need to create a document so we can access the inner html:

```csharp
webBrowser.Navigate("about:blank");
```

Now that we have a document, we need to populate it with something.

We can use the Document property of the WebBrowser control to manipulate its contents. First we must cast it to an IHTMLDocument2:

```csharp
mshtml.IHTMLDocument2 CurrentDocument = (mshtml.IHTMLDocument2)webBrowser.Document;
```


Now lets populate it with some HTML and a couple tags that we can programatically fill in: 

```csharp
CurrentDocument.writeln(
        @"<html>
                        <body>
                            <TABLE id='Table1' cellSpacing='0' cellPadding='0' 
                                    width='400' border='1' borderColor='black'>
                                <TR>
                                    <TD style='BORDER:0;WIDTH: 86px'>
                                        <DIV style='DISPLAY: inline; WIDTH: 150px; 
                                                HEIGHT: 15px' >Name:</DIV>
                                    </TD>
                                    <TD style='BORDER:0;'>
                                        <div id=txtName> </div>
                                    </TD>
                                </TR>
                                <TR>
                                    <TD style='BORDER:0;WIDTH: 86px'>
                                        <DIV style='BORDER:0;DISPLAY: inline; WIDTH: 70px; 
                                                HEIGHT: 15px' >Description:</DIV>
                                    </TD>
                                    <TD style='BORDER:0;'>
                                        <div id=txtDesc> </div>
                                    </TD>
                                </TR>
                            </TABLE>
                        </body>
                    </html>");
```


Lets fill in the txtName and txtDesc DIV elements with user input data:

```csharp
mshtml.HTMLDocument CurrentDocument = (mshtml.HTMLDocument)webBrowser.Document;

((mshtml.HTMLDivElement)CurrentDocument.all.item("txtName",0)).innerText = txtName.Text;
((mshtml.HTMLDivElement)CurrentDocument.all.item("txtDesc",0)).innerText = txtDescription.Text;
```


That was easy! :)