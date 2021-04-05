Now that you know all the needed Spring basics, let’s focus on web apps and how you use Spring to implement them. You can use all the Spring capabilities we discussed until now to implement any kind of app. But often, with Spring, the applications you will implement are web apps. In chapters 1 through 6, we discussed the Spring context and aspects which are mandatory for understanding what comes next in the book (including what you’ll find in this chapter). If you jumped directly to this chapter, and you don’t know yet how to work with the Spring context and aspects, you might find our discussion difficult to understand. I strongly recommend you first make sure you know the basics of using the framework before going further.

Spring makes web app development straightforward. We’ll start this chapter in section 7.1 by discussing what web apps are and how they work.

To implement web apps, we’ll use a project of the Spring ecosystem named Spring Boot. In section 7.2, we’ll discuss Spring Boot and why it’s essential to use it in apps implementations. In section 7.3, we’ll discuss the standard architecture of a simple Spring web app, and we’ll implement a web app using Spring Boot. At the end of this chapter, you’ll understand well how a web app works, and you’ll be able to implement a basic web app with Spring.

This chapter's main purpose is to help you understand the foundation that supports web apps' implementation. In chapters 8 and 9, we will implement the major capabilities you find in most web apps in production. But everything we discuss in these next chapters relies on the foundation you learn by reading this chapter. Moreover, in chapter 10, we discuss REST services and implementing REST endpoints. You’ll also find that behind REST endpoints in Spring stay the same foundation we discuss in this chapter.

## 7.1    What is a web app?

In this section, we take a look at what a web app is. I’m sure you use web apps daily. Probably you just left a few tabs open in a web browser before starting to read this chapter. Maybe you don’t even read this book on paper, but you use the Manning liveBook web app for this.

Any app you access through your web browser is a web app. Years ago, we used desktop apps installed on our computers for almost anything we were doing (figure 7.1). With time, most of these apps became accessible via a web browser. Accessing an app in a browser makes the app more comfortable to use. You don’t have to install anything, and you can use it from any device that has access to the Internet, such as a tablet or smartphone.

Within this section, I want to make sure you have a clear overview of what we’re going to implement. What is a web app, and what do we need to build and execute such an app? Once you have a clear picture of a web app, we continue implementing one with Spring.

### 7.1.1   A general overview of a web app

In this section, we take a high-level look at what a web app is from the technical standpoint. This overview allows us to discuss in further detail our options for creating web apps.

First of all, a web app is composed of two parts:

-   The client-side, which is what the user directly interacts with. A web browser represents the client-side of a web app. The browser sends requests to a web server, receives responses from the web server, and provides a way for the user to interact with the app. We also refer to the client-side of a web app as the frontend.
-   The server-side, which receives requests from the client and sends back data in response. The server-side implements logic that processes and sometimes stores the client request's data before sending a response. We also refer to the server-side of a web as the backend.

   Figure 7.2 presents the big picture of a web app.
   
   When discussing web apps, we usually refer to a client and a server. But it’s important to keep in mind that the backend side of a web app serves multiple clients concurrently. Numerous people may use the same app at the same time on different platforms. Users can access a browser to use your app on a computer or another system (a phone, a tablet, and so on) (figure 7.3).
   
   ### 7.1.2   Different fashions of implementing a web app with Spring

In this section, we discuss the two main designs you can use to implement a web application. We’ll implement apps in both these ways in chapters 8 through 10, and we’ll discuss the implementation details when we go deeper into implementing each. But for now, I want you to be aware of your choices and have a general understanding of these options. It’s important to know how you can create your web app to avoid getting confused later when implementing examples.

We classify the approaches of creating a web app as

1.  Apps where the backend provides the fully baked view in response to a client’s request. The browser directly interprets the data received from the backend and displays this information to the user in these apps. We discuss this approach and implement a simple app to prove this approach in this chapter. We continue then our discussion with more complex details relevant to production apps in chapters 8 and 9.
2.  Apps using frontend-backend separation. For these apps, the backend only serves raw data. The browser doesn’t display the data in the backend’s response directly. The browser runs a separate frontend app that gets the backend responses, processes the data, and instructs the browser what to display. We discuss this approach and implement examples of it in chapter 9.

Figure 7.4 presents the first approach in which the app doesn’t use a frontend-backend separation. For these apps, almost everything happens on the backend side. The backend gets requests representing user actions and executes some logic. In the end, the server responds with what the browser needs to display. The backend responds with the data in formats that the browser can interpret and display, such as HTML, CSS, images, and so on. It can also send scripts written in languages that the browser can understand and execute (such asJavaScript).

In figure 7.5, you find the way an app using frontend-backend separation works. Compare the server's response in figure 7.5 with the response the server sends back in figure 7.4. Instead of telling the browser precisely what to display, the server now only sends raw data. The browser runs an independent frontend app it loads at an initial request from the server. This frontend app takes the server's raw response, interprets it, and decides how the information is displayed. We’ll discuss more details about this approach in chapter 9.

You will find both these approaches in production apps. Sometimes developers refer to the frontend-backend separation approach as being a modern approach. The separation of frontend end backend helps in making the development easier to manage for larger apps. Different teams take the responsibility of implementing the backend and frontend, allowing more developers to collaborate easier to develop the apps. Also, the deployment of the frontend and the backend can be independently managed. For a larger app, this flexibility is also a nice benefit.

The other approach where the app doesn’t use frontend-backend separation is today used mostly for small apps. After discussing in detail both approaches in this chapter and chapters 8 through 10, I’ll teach you the advantages of both these methods, and you’ll know when to choose an approach based on your app’s needs.

### 7.1.3   Using a servlet container in web apps development

In this section, we analyze more deeply what and why you need to build a web app with Spring. Up to now, in this chapter, we’ve seen that a web app has a frontend and a backend. But we didn’t discuss explicitly implementing a web app with Spring. Of course, our purpose is to learn Spring and to implement apps with Spring. So we have to take a step forward and find out what we need to implement web apps with the framework.

One of the most important things to consider is the communication between the client and the server. A web browser uses a protocol named Hypertext Transfer Protocol (HTTP) to communicate with the server over the network. This protocol accurately describes how the client and the server exchange data over the network. But unless you are passionate about networking, you don’t need to understand how HTTP works in detail to write web apps. As a software developer, you’re expected to know that the web apps components use this protocol to exchange data in a request-response fashion. The client sends a request to the server, and the server responds. The client waits for the response after every request it sends. In appendix C, you find all the details you need to know about HTTP to understand the discussion in chapters seven through nine properly.

But does that mean your app needs to know how to process the HTTP messages? Well, you can implement this capability if you wish, but unless you want to have some fun writing low-level functionalities, you’ll use a component already designed to understand HTTP.

In fact, what you need is not only something that understands HTTP. You also need something that can translate the HTTP request and response to a Java app. This “something” is a servlet container. The servlet container (sometimes referred to as a web server) is a translator of the HTTP messages for your Java app. This way, your Java app doesn’t need to take care of implementing the communication layer. One of the most appreciated servlet container implementations is Tomcat. Tomcat is also the dependency we’ll use for the examples in this book.

##### NOTE

We use Tomcat for the examples in this book, but you can use Tomcat's alternatives for your Spring app. The list of solutions used in real-world apps is long. Among these, you find Jetty (https://www.eclipse.org/jetty/), JBoss (https://www.jboss.org/), and Payara ([https://www.payara.fish/](https://www.payara.fish/)).

In figure 7.6, you find a visual representation of a servlet container (Tomcat) in our app’s architecture.

But if this is everything a servlet container does, why we name it “servlet” container? What is a servlet? A servlet is nothing more than a Java object which directly interacts with the servlet container. When the servlet container gets an HTTP request, it calls a servlet object's method providing the request as a parameter. The same method also gets a parameter representing the HTTP response used by the servlet to set the response sent back to the client that made the request.

Some time ago, the servlet was the most critical component of a backend web app from the developer’s point of view. Suppose a developer had to implement a new page accessible at a specific path in the URL (for example /home, /profile/edit, and so on) for a web app. The developer needed to create a new servlet instance and configure it in the servlet container assigning it to a specific path (figure 7.7). The servlet contained the logic associated with the user’s request and the ability to prepare a response, including info for the browser on how to display the response. For any path the web client could call, the developer needed to add the instance in the servlet container and configure it. Because such a component manages servlet instances you add into its context, we name it a servlet container. A servlet container has basically a context of servlet instances it controls, just as Spring does with its beans. For this reason, we call a component such as Tomcat a “servlet container.”

As you’ll learn further in this chapter, we don’t usually create servlet instances anymore. We’ll use a servlet with the Spring apps we develop with Spring, but you won’t need to write this servlet yourself. So you don’t have to focus on learning to implement servlets, but you need to remember the servlet is the entry point to your app’s logic. It’s the component the servlet container (Tomcat in our case) directly interacts with. It’s how the request data enters your app and how the response goes through Tomcat back to the client.

## 7.2    The magic of Spring Boot

All right, so to create a Spring web app, we need to configure a servlet container, we need to create a servlet instance, and then we need to make sure we correctly configure this servlet instance such that Tomcat calls it for any client request. What a headache was to write so many configurations! Oh, trust me, writing these configurations was a headache. Many years ago, when I was teaching Spring 3 (the latest Spring version of that time), and we configured web apps, this was the part both the students and I hated the most. Fortunately, times changed, and today I don’t have to bother you by teaching such configurations.

In this section, we’ll discuss Spring Boot, a tool for implementing modern Spring apps. Spring Boot is now one of the most appreciated projects in the Spring ecosystem. Spring Boot helps you create Spring apps more efficiently and focus on the business code you write by eliminating a huge part of the code you used to write for configurations. Especially in a world of service-oriented architectures (SOA) and microservices, where you create apps more often (remember the discussion in appendix A), avoiding the pain of writing configurations is helpful.

I consider the most critical features Spring Boot offers to be

-   Simplified project creation – You can use a project-initialization service to get an empty but configured skeleton app.
-   Dependency starters – Spring Boot groups certain dependencies used for a specific purpose with dependency starters. You don’t need to figure out anymore all the must-have dependencies you need to add to your project for one particular purpose and neither which versions you should use for compatibility.
-   Autoconfiguration based on dependencies - Based on the dependencies you added to your project, Spring Boot defines some default configurations. Instead of writing yourself all the configs, you know only need to change the ones provided by Spring Boot, which don’t match what you need for your app. Changing the configs requires most likely less code (if any).

Let’s discuss these two essential features of Spring Boot more in-depth and also apply them within an example. This first example of this chapter is the first Spring web app we write.