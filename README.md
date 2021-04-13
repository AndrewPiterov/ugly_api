# Ugly API

![alt text](https://github.com/AndrewPiterov/ugly_api/blob/main/screenshots/0_cover_final.jpg "Ugly API")

In this article, I’d like to talk about the problems I faced while integrating an API for the HTTP protocol and share my experience in solving them.

**Table Of Contents:**

* Intro
* REST vs Non REST architecture
* Ignoring Header Accept: application/json
* Mixing JSON keys case types
* Different response to the same request
* Conclusion

## Intro

When developing a frontend for Mobile/Web apps, you often come across the fact that the backend has not been implemented yet. So, you have to wait for the developer on the backend when he provides the necessary requests persistently drawing his attention to yourself. And another situation when the necessary http requests have already existed, but they are very poorly and crookedly implemented is far from being unusual.

Maybe, I wouldn’t have written this article. But the fact that all the examples of poor API implementation I’m giving below came across to me in the same project and at the same time seemed very impressive to me!

In this project, I am developing a mobile app in Flutter using the Retrofit package, which helps me save time and reduce code and I have to write myself generating significant code automatically. And I use Insomnia for the initial verification of requests before implementing them in the code as well.

Before I show the issues that I faced, I would like to mention that before I became a mobile developer I have been working as a Full-Stack developer for more than 5 years. And I understand how important it is to implement beautiful API for many frontend clients, with which it is easy and pleasant to integrate and safe, in case of future changes in the API.

Video version available on YouTube on Russian

<a href="http://www.youtube.com/watch?feature=player_embedded&v=oj-i1IBejcI" target="_blank"><img src="http://img.youtube.com/vi/oj-i1IBejcI/0.jpg" alt="Ugly API" width="240" height="180" border="10" /></a>

So, let’s start.

## REST vs Non REST architecture
The fact that the API architecture is not implemented in the RESTful style was the first disappointment. To be honest, I don’t remember when I had to face such a situation.

REST stands for **RE**presentational **S**tate **T**ransfer. RESTful is a sort of API architecture implementation that makes use of the HTTP protocol in the best way possible. With REST, we need to think about the app in terms of resources and to determine which resources we want to expose to the outside world (for example, tasks, customers, etc.). Here we use the verbs defined by the HTTP protocol to perform CRUD operations on these resources, such as GET, POST, PUT, DELETE, etc.


Here is a RESTful API example:

<image>

My project’s API architecture that is non-RESTful architecture looks like this:

<image>

You can see that all requests are presented in the same type — these are POST requests. And each request must contain a type parameter that defines the operation. Apparently, on the server, in this single endpoint, there are some if-else or switch-statements for checking this parameter.

Naturally, I prefer RESTful, but since the other implementation works properly and it is possible to more or less configure requests with Retrofit on the client, I accepted this fact and continued to integrate with the API.

## Ignoring Header Accept: application/json

Usually, I first check a request to API in Insomnia and then implement them in my code.

After I made a request in Insomnia, I saw a nice json in the Preview tab and thought that a json object was coming to me. I formalized all this in my code; Retrofit automatically converted response to the request to a Dart class for me, and everything worked great.

But the next day, my app stopped working due to an error related to the request to the server. The error text was: “An error when converting the line to an object occurs”.

I went back to Insomnia again and checked the request. I saw the same json as before in the Preview tab. And only after checking the Header of the request, I found that the Content-Type has changed, and its value has already been text/html, charset-utf-8, although the Preview shows me json.

Thus, I determined that the response type from the server changed from application/json to text/html due to which Retrofit can no longer convert the response from the server of the line type automatically to a Dart object.

I decided to try using Accept Header in a request that will tell the server something that “I’m a client expecting a json response from you”. But it didn’t work because the server doesn’t consider this header.

<https://cdn.statically.io/screenshot/:url>

And again, I had to add even more code to work around this server response type issue.

1. Added a new version of the request to Retrofit where I expect a response as “String” type in the second version

<image>

2. Then I started initiating a new version of the query that returns a String and added a conversion String to object. Meanwhile, Retrofit could do this conversion itself in the code it generated. And now, in every place where we make requests to the server, we must add this conversion ourselves to receive responses from it:

<image>

As a result, even more code lines were added, which is not encouraging.


## Mixing JSON keys case types

There are several types of formatting field names in json:

<image>

In an ideal scenario, you need to choose one option you like and comply with it during the whole project and in all requests. But in my project, the backend developer decided to mix several types:

<image>

In an ideal scenario, the camelCase style is used for naming class fields and methods in Dart. And if json comes from the server with fields in camelCase style, then Retrofit will automatically, without any effort, correctly match the class and json fields. But in this situation, I have to specify additional JsonKey annotations, which are as snake_case in one case, and as UPPER_CASE_SNAKE_CASE in the other one:

<image>

As a result, there is no uniformity; if you make a mistake in the case type, you can lose the meaning since the fields will not match. I got even angrier at the API, but the integration with the API could still be continued with more effort and even more code.

## Different response to the same request

Here I almost got heart attack… When I tried to log into the app as a different user, my app broke again. I got the following error: “Could not convert String to Int”. This was due to the fact that my class describing the response from the server became incorrect. The types described in the class don’t correspond to the types of fields available in the json response. I opened Insomnia again and ran the same login request for two different users. And I found that the numbers come as Strings in response in one case, and they come as numbers in the other one. 🤯 You can see that in Insomnia, the lines are highlighted in yellow, and the numbers are in purple:

<image>

How can it be that the same json arrives for the same request, but with different types of values?

<image>

I know the server was written in PHP. Folks, who encode in PHP, could you write in the comments how is this possible?

## Conclusion

Currently, I am experiencing the opposite feelings. On the one part, I’m upset that I have to integrate such an ugly server, and on the other part, I’m already looking forward to the next case that will show how else you can disfigure the API.

I’d like to know about your experience in integrating with the API, and what problems you faced.

Thank you in advance. Happy coding! 👨🏻‍💻