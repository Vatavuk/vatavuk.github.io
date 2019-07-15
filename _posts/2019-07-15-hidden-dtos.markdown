---
layout: post
title: Elegant Business Logic
date: 2018-08-15 13:32:20 +0300
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: i-rest.jpg # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Java, OOP]
---

My New Year's resolution came early this year. I will never use data transfer objects in my java
applications again. No getters and setters anymore, thank you. It was not a pleasure and you wont be missed.
My pain is not unique, plenty has been said about dtos and getters and setters. 
They separate data from behaviour and they break encapsulation moving design away from OOP paradigm.

I would like to show you how you can ...
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


There are already a few solutions that are worth "checking". 
One approach is to use arbitrary data structure, such as JsonObject,
instead of plain java dto to pass data around. In order to access data we
animate it, expose it through an interface. The point is that if data is changed
we don't need to update interfaces. The problem with this method is that
JsonObject is still a dto and we can access it's internals nonetheless.
   
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

Let's change the signature a bit: 

"""
class Patient {
    
    private String ssn, address;
    // stuff
    <T> T print(Function<PatientContent, Printable<T> media) {
        return media.apply(new PatientContent(ssn, address)).print();
    }
}
"""
It looks a little bit weird but follow me through. Print method now accepts a function


