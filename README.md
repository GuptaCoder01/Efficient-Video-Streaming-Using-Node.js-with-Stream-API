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

Getting started
First, we’ll set up a new folder and initialize npm:

npm init
Now, install Express and nodemon:

npm install --save express nodemon



Given that your video element is an empty folder, you’ll need to generate an HTML file as follows:


<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Video Streaming With Node</title>
    <style>
        /* General page styling */
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
            background-color: #130e0e;
            color: #333;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
        }

        h1 {
            font-size: 2rem;
            margin-bottom: 20px;
            text-align: center;
            color: #e6e0e0;
        }

        /* Video player styling */
        #videoPlayer {
            border: 2px solid #444;
            border-radius: 10px;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
            background-color: #000;
        }

        /* Responsive adjustments */
        @media screen and (max-width: 768px) {
            #videoPlayer {
                width: 90%;
            }

            h1 {
                font-size: 1.5rem;
            }
        }
    </style>
</head>

<body>
    <h1>Streaming Video in CHUNKS With Node.js</h1>
    <video id="videoPlayer" width="50%" controls muted="muted" autoplay>
        <source src="/video" , type="video/mp4" />
    </video>
</body>

</html>



Writing the /video endpoint

const express = require("express");
const app = express();
const fs = require("fs");

app.get("/", function (req, res) {
    res.sendFile(__dirname + "/index.html");
});

app.get("/video", function (req, res) {
    const range = req.headers.range;
    if (!range) {
        res.status(400).send("Requires Range header");
    }
    const videoPath = "Chris-Do.mp4";
    const videoSize = fs.statSync("Chris-Do.mp4").size;
    const CHUNK_SIZE = 10 ** 6;
    const start = Number(range.replace(/\D/g, ""));
    const end = Math.min(start + CHUNK_SIZE, videoSize - 1);
    const contentLength = end - start + 1;
    const headers = {
        "Content-Range": `bytes ${start}-${end}/${videoSize}`,
        "Accept-Ranges": "bytes",
        "Content-Length": contentLength,
        "Content-Type": "video/mp4",
    };
    res.writeHead(206, headers);
    const videoStream = fs.createReadStream(videoPath, { start, end });
    videoStream.pipe(res);
});

app.listen(8000, function () {
    console.log("Listening on port 8000!");
});
