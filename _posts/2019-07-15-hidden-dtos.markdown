---
layout: post
title: Elegant Business Logic
date: 2018-08-15 13:32:20 +0300
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: i-rest.jpg # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Java, OOP]
---

My New Year's resolution came early this year. I will never use data transfer objects in java
applications again. No getters and setters anymore, thank you. It was not a pleasure and you wont be missed.


My pain is not unique, plenty has been said about dtos and getters and setters. 
They break encapsulation and by using it we separate data from behaviour which moves our design away from OOP paradigm.

I would like to show you a concrete example how you can manipulate data without exposing it to
the rest of the codebase. 

The goal is to clean up our business logic/use cases from unnecessary data exposure. What does it mean?
Let's take a look at the following data class:

'''java
class PatientDto {

    private String ssn;
    private String name;
    private String surname;
    private String address;
    private String nurse;
    
    //getters and setters...
} 
'''

If the use case only uses ssn to verifies patient uniqueness...


   
We have also concept of printers, to let object print itself. 

'''java
class Patient {
    private String ssn, address;
    // stuff
    void print(Media media) {
        media.write("ssn", ssn);
        media.write("address", address);
    }
}
// printing to html
patient.print(htmlForm);
Html html = htmlForm.toHtml();
'''
Only patient object has the information about patient data and it can print 
itself to various media (html, xml, json...). If data changes, only one class
needs to be changed. This is a very powerful OOP tehnique but it has some
drawbacks. What and how.
What if we have additional requirement which ...
  
I propose to give the printer pattern some steroids to let user take full control
of what data should be printed and how.

Let's change the signature a (bit - duplication): 

"""java
class Patient {
    
    private String ssn, address;
    // stuff
    <T> T print(Function<PatientContent, Printable<T> media) {
        return media.apply(new PatientContent(ssn, address)).print();
    }
}
"""

It looks weird but follow me through. Print method now accepts a function


"""java
public class PatientContent
{
    protected final String ssn;
    
    protected final String address;

    public PatientContent(PatientContent content) {
        this(content.ssn, content.address);
    }

    public PatientContent(String ssn, String address)
    {
        this.dto = dto;
    }
}
"""

"""java
public class JsonPatientPrintout extends PatientContent implements Printable<JsonObject>
{
    public JsonPatientPrintout(PatientContent content)
    {
        super(content);
    }

    @Override
    public JsonObject print()
    {
        return Json.createObjectBuilder().
            .add("ssn", this.ssn)
            .add("address", this.address)
            .build();
    }
}
"""

Now we print the patient to json:
"""java
JsonObject json = patient.print(JsonPatientPrintout:new);
"""

Let's play a little bit more and 
The fun part is that there are no harm in using the dto in such a manner. It cannot be used outside
of a printer context. Getters cannot leak to other parts of the application. You don't even need to
hold PatientContent in a separate package to avoid package access because no one other than printer can
get PatientContent instance.
Bear in mind that such a dto should absolutely be immutable, without exceptions. You don't want your derived class
to mutate the dto state. 

Full example can be find here: 

