---
title: "How to retrieve a collection of attributes (LINQ to XML) (C#)"
ms.date: 07/20/2015
ms.assetid: a49ee7a3-b2c2-4d49-9b5c-b7c6c41f4f13
---
# How to retrieve a collection of attributes (LINQ to XML) (C#)
This topic introduces the <xref:System.Xml.Linq.XElement.Attributes%2A> method. This method retrieves the attributes of an element.  
  
## Example  
 The following example shows how to iterate through the collection of attributes of an element.  
  
```csharp  
XElement val = new XElement("Value",  
    new XAttribute("ID", "1243"),  
    new XAttribute("Type", "int"),  
    new XAttribute("ConvertableTo", "double"),  
    "100");  
IEnumerable<XAttribute> listOfAttributes =  
    from att in val.Attributes()  
    select att;  
foreach (XAttribute a in listOfAttributes)  
    Console.WriteLine(a);  
```  
  
 This code produces the following output:  
  
```output  
ID="1243"  
Type="int"  
ConvertableTo="double"  
```  
  
## See also

- [LINQ to XML Axes (C#)](./linq-to-xml-axes-overview.md)
