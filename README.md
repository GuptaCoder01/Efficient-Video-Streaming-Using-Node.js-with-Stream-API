When building web apps, developers frequently have to handle different types of media, some of which can be complex. In this article, we’ll create our own video streaming server using Node.js.

![image](https://github.com/user-attachments/assets/4b3c565a-31d0-4142-bff6-b44b7d00918a)

# Project overview
Before we begin coding our project, let’s review how our app will work at a high-level. In the image above, the browser is on the left and the server is on the right. On your site, you’ll have an HTML5 video element with a source that points to the /video endpoint.

First, the video element makes a request to the server, then the header provides the desired range of bytes from the video. For example, at the beginning of the video, the requested range would be from the 0th byte onwards, hence the 0-. The server will respond with a 206 HTTP status, indicating it is returning partial content with the proper header response, which includes the range and content length.

The response headers indicate to the video element that the video is incomplete. As a result, the video element will play what it has downloaded so far. When this happens, the video element will continue making requests, and the cycle will continue until there are no bytes left.

Application pros and cons
Now that we understand how our app will work, let’s consider some of the pros and cons of following this methodology.

As you may have guessed from the application overview, our streaming server will be fairly simple to implement. Essentially, we’re creating a file system and returning it back to the client. Our server will allow us to select timeframes throughout the video and decide how big of a payload to send back. For mine, I chose 1MB, but you have the freedom to play around with it.

However, because of our app’s simplicity, the server and video player don’t work as well together as we would like. Essentially, the video player will just request the part of the video you’re on, without taking into account what you already requested. It’s likely that you’ll end up requesting some of the same resources over and over again.

> Getting started
First, we’ll set up a new folder and initialize npm:

> npm init
> Now, install Express and nodemon:

> npm install --save express nodemon




