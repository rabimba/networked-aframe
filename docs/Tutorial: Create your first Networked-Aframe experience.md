# How to build your first multi-user virtual reality experience (on the web)


Hello friends! This tutorial will show you how to write your very first multi-user virtual reality experience on the web.

You might be thinking: wait you can do VR on the web? And the answer is yes! Using the [WebVR standard](https://webvr.info/) and an awesome library called [A-Frame](https://aframe.io), VR on the web is actually really easy and evolving rapidly. Your second question is then: wait to do multi-user won't I have to know all about servers and complicated networking protocols? Answer: not anymore! The recently released [Networked-Aframe](https://github.com/haydenjameslee/networked-aframe) (NAF for short) has you covered. NAF hides away a lot of the networking complexity, allowing you to write multi-user VR apps with a few lines of HTML and Javascript.

So, let's dive in:

## Required Tools

We'll need the following tools:

- NodeJS
- NPM

If they aren't already installed, follow these quick tutorials to get started: [NodeJS for Windows](https://nodesource.com/blog/installing-nodejs-tutorial-windows/) or [NodeJS for MacOS](https://nodesource.com/blog/installing-nodejs-tutorial-mac-os-x/). NPM will be installed by default when you install NodeJS via these methods.


## Install Dependencies

Create a new folder called `naf-tutorial` and open it.

Now let's setup the required dependencies. Create a file called `package.json` and add the following text:

```json
{
  "name": "naf-tutorial",
  "version": "1.0.0",
  "description": "My first multi-user virtual reality",
  "scripts": {
    "start": "node ./server/server.js"
  },
  "author": "YOUR_NAME",
  "dependencies": {
    "networked-aframe": "^0.1.0"
  }
}
```

This file is how NPM knows which dependencies to install and will let us use some handy shortcuts later on.

Now let's install the dependencies. Open up the command line prompt of your choosing and navigate to your new `naf-tutorial` folder. Then run:

```sh
npm install
```

You'll notice a new folder called `node_modules` which contains all the dependencies for networked-aframe.


## Setup the Server

Let's copy the example server that comes with networked-aframe into our project. From your project's root, run the respective command for your OS:

```sh
# MacOS
cp -r ./node_modules/networked-aframe/server/ ./server/

# Windows
robocopy .\node_modules\networked-aframe\server\ .\server\ /e
```

You'll now see another new folder: `server/`. Inside it you'll see the `static` folder which contains the networked-aframe examples. You'll also see `server.js` which is the the server for managing the network connections and hosting the static files.

Let's run the server and start playing with the examples. From your project's root run:

```sh
node ./server/server.js
```

You'll notice the `package.json` also has a shortcut to start the server that you can use by running:

```sh
npm start
```

Now that the server's running you can fire up your favorite web browser (as long as its the latest Chrome or Firefox) and enter this URL:

```
http://localhost:8080
```

You should see an index page that shows a list of the included examples.

<img src="http://i.imgur.com/pc07Nyir.png" width="500">

To see the networking in action, open up two browser windows and go to the same example on both. Click and hold with the mouse to move your avatar's head around. The other browser should see your avatar's movements.

<img src="http://i.imgur.com/B5kbgXg.gif" width="500">

By the way, if at any point in this tutorial you don't see the same results, or you run into an unknown error, don't hesitate from posting a question on the [networked-aframe issues page](https://github.com/haydenjameslee/networked-aframe/issues). I'm sure other people will run into the same issues and greatly appreciate the conversation your question will spark.


## Create an Example from Scratch

So we've seen how to get your own local server up and running with the default examples. Let's have a go at making our own example from scratch.

In the `naf-tutorial/server/static/` folder, create a new file and name it `my-example.html`.

Here's the template we'll start with:

```html
<html>
  <head>
    <script src="https://aframe.io/releases/0.5.0/aframe.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/socket.io/1.4.5/socket.io.min.js"></script>
    <script src="easyrtc/easyrtc.js"></script>
    <script src="https://unpkg.com/networked-aframe/dist/networked-aframe.min.js"></script>
  </head>
  <body>
    <a-scene></a-scene>
  </body>
</html>
```

We can see that all the NAF dependencies are included; aframe, socket.io, EasyRTC and Networked-Aframe itself. Copy this template into `my-example.html`. To check the dependencies are setup correctly, start the server by running `npm start` and then head to `localhost:8080/my-example.html`. You should see a blank white page, and no Javascript errors in the Developer Console. There'll probably be a bunch of mumbo jumbo in the dev console showing the current versions of each library.

First up we need to add the `network-scene` component to the A-Frame `a-scene` tag.

Modify the `<a-scene>` tag like so:

```html
<a-scene network-scene="
  app: myApp;
  room: room1;
  audio: false;
  debug: true;
"></a-scene>
```

Now when you open two tabs at `localhost:8080/my-example.html` you should see the following logs, indicating a data channel has been established between the two clients:

<img src="http://i.imgur.com/IpqHRGQ.png" width="500">

To see the full list of options your app with other options check out the documentation on the [network-scene component](https://github.com/haydenjameslee/networked-aframe#scene-component).


## Create an Avatar

NAF uses a template system to make it easy to define objects to sync across all users. One common template that a lot of apps want is an avatar. NAF has a convenience function for making avatars that automatically follow your head movements, along with a couple of other little features. Its the next thing we'll add to our scene.

Add a script tag in `my-example.html`'s `<head>` section like so:

```html
<head>
  ...
  <script>
    function onConnect () {
      var position = '0 1.3 0';
      var rotation = '0 0 0';
      NAF.entities.createAvatar('#avatar-template', position, rotation);
    }
  </script>
</head>
```

Wait a second where did this `onConnect` function come from and what does it want? `onConnect` is called by NAF when you have successfully connected to the server. At this point you will start making connections to other users in the room and you are able to create network entities (entities that are synced to other users). You can change the name from `onConnect` to another function name in the [network-scene component](https://github.com/haydenjameslee/networked-aframe#scene-component). The variables `position` and `rotation` specify the position and rotation that the avatar will be spawned at.

Back to templates, you'll notice the first argument to `createAvatar` is `'#avatar-template'`. This is specifying the HTML ID of a template to use as the avatar. In order for this to work we need to add the template to the scene.

Templates are added in A-Frame's `<a-assets>` tag. Let's add the assets tag with a simple avatar at the top of the `a-scene` tag:

```html
<a-assets>
  <script id="avatar-template" type="text/html">
    <a-entity class="avatar">
      <a-sphere class="head"
        color="#5985ff"
        scale="0.45 0.5 0.4"
      ></a-sphere>
      <a-entity class="face"
        position="0 0.05 0"
      >
        <a-sphere class="eye"
          color="#efefef"
          position="0.16 0.1 -0.35"
          scale="0.12 0.12 0.12"
        >
          <a-sphere class="pupil"
            color="#000"
            position="0 0 -1"
            scale="0.2 0.2 0.2"
          ></a-sphere>
        </a-sphere>
        <a-sphere class="eye"
          color="#efefef"
          position="-0.16 0.1 -0.35"
          scale="0.12 0.12 0.12"
        >
          <a-sphere class="pupil"
            color="#000"
            position="0 0 -1"
            scale="0.2 0.2 0.2"
          ></a-sphere>
        </a-sphere>
      </a-entity>
    </a-entity>
  </script>
</a-assets>
```

Refresh both browser tabs and move with the arrow keys. Once connected you should see an avatar appear for the other user.

The avatars appear in the exact same position: '0 1.3 0' which is not a great experience when in social VR. Virtual Personal Space is a thing! So next step is to choose a better spot to spawn the avatar. At the moment its a little tricky to know where other users are before you've spawned the avatar (since you currently don't know when you've connected to all users), however we can improve the experience a lot by choosing a random point on a circle, and praying to [RNGesus](https://cdn.mmos.com/wp-content/gallery/rng-isnt-random/RNG-meme.jpg) that the random number generator doesn't spawn you close so other users. If you're looking for better support for spawn collision avoidance [let me know](https://github.com/haydenjameslee/networked-aframe/issues) and we'll work something out.

Modify the existing script tag with the code for spawning in a circle:

```html
<script>
  function onConnect () {
    // Get random angle
    var angleRad = Math.random()*Math.PI*2;

    // Get position around a circle
    var position = randomPointOnCircle(3, angleRad);
    var positionStr = position.x + ' 1.3 ' + position.y;

    // Get rotation towards center of circle so avatars can see each other
    var angleDeg = angleRad * 180 / Math.PI;
    var angleToCenter = -1 * angleDeg + 90;
    var rotationStr = '0 ' + angleToCenter + ' 0';

    // Create avatar with this position and rotation
    NAF.entities.createAvatar('#avatar-template', positionStr, rotationStr);
  }

  function randomPointOnCircle(radius, angleRad) {
    x = Math.cos(angleRad)*radius;
    y = Math.sin(angleRad)*radius;
    return {x: x, y: y};
  }
</script>
```

Refresh those browsers and your scene should be starting to take shape (albeit with a [completely white environment](https://c1.staticflickr.com/8/7405/11599367004_2f03c315fb_b.jpg)).

## Environment

To spruce up your world add the following HTML tags:

```html
<!-- Add to bottom of the a-assets tag -->
<img id="grid" src="https://img.gs/bbdkhfbzkk/stretch/https://i.imgur.com/25P1geh.png" crossorigin="anonymous">
<img id="sky" src="https://img.gs/bbdkhfbzkk/2048x2048,stretch/http://i.imgur.com/WqlqEkq.jpg" crossorigin="anonymous" />

<!-- Add to bottom of a-scene tag -->
<a-entity position="0 0 0"
  geometry="primitive: plane; width: 10000; height: 10000;" rotation="-90 0 0"
  material="src: #grid; repeat: 10000 10000; transparent: true; metalness:0.6; roughness: 0.4; sphericalEnvMap: #sky;"></a-entity>

<a-entity light="color: #ccccff; intensity: 1; type: ambient;" visible=""></a-entity>
<a-entity light="color: #ffaaff; intensity: 1.5" position="5 5 5"></a-entity>

<a-sky src="#sky" rotation="0 -90 0"></a-sky>
```

Things should be looking a little more Tron-like now.

<img src="http://i.imgur.com/DVVIku1.png" width="500">

At this point your example will be very close to the included `basic.html` example, so if you've run into any problems check out the [basic example source code](https://github.com/haydenjameslee/networked-aframe/blob/master/server/static/basic.html) for a working reference.

## Advanced Features

Now you have a basic networked WebVR scene up and running. But there's a lot more that Networked-Aframe can do. The next section will take a look at some of the advanced features that give you a lot more control over your experience.

### Audio

NAF has built in voice chat, and all you need to do to enable it is modify the `network-scene` component. Change the audio property to `true` and your users can speak to each other! This is a little hard to test locally because the audio feedback will destroy your ears, so try it with headphones and you'll hear your voice being echoed back to you without the feedback. Note: in order for audio streaming to work on a hosted server you'll need to be using HTTPS. I'm planning on writing a follow-up tutorial to this one that will explain how to deploy NAF to a live server, including how to setup HTTPS really easily using [Certbot](https://certbot.eff.org/).


### Syncing Custom Components

Components are synchronized by checking the `data` property [provided by A-Frame](https://aframe.io/docs/0.5.0/core/entity.html#getattribute-componentname) on a network 'tick'. How quickly this tick happens can be defined in the [NAF Options](https://github.com/haydenjameslee/networked-aframe#options), but the default is 15 times per second. On each tick the `data` property is checked against its previous value, and if it changed it's sent over the network to the other users.

So how do we choose which components to sync? By default, the `position` and `rotation` components are synced but NAF lets you specify any component that you wish to sync, included child components found in the deep depths of your templates.

To define synced components we introduce the concept of a NAF `schema`. These schemas are basic Javascript objects that either define just the component's name if the component is at the root level of the entity, or they define a CSS selector and the component name if the component is on a child entity.

Here's an example schema that syncs the position and rotation of the root entity, while syncing the color of the avatar's head:

```Javascript
var avatarSchema = {
  template: '#avatar-template',
  components: [
    'position',
    'rotation',
    {
      selector: '.head',
      component: 'material'
    }
  ]
};
NAF.schemas.add(avatarSchema);
```

In the `basic.html` example this is used in combination with [ngokevin's Randomizer component](https://github.com/ngokevin/kframe/tree/master/components/randomizer) to give each avatar's head a synced randomized color.

To add the randomizer component include this script in the `<head>` section:

```html
<script src="https://unpkg.com/aframe-randomizer-components@^3.0.1/dist/aframe-randomizer-components.min.js"></script>
```

and change the avatar template's head tag to use the `random-color` component:

```html
<a-sphere class="head"
  scale="0.45 0.5 0.4"
  random-color
></a-sphere>
```

### Deleting entities

In order to delete a network entity the user that created the entity can simply delete the HTML element from the scene, by regular HTML deletion methods. NAF then handles syncing the deletion under the hood. Currently only the user who created the entity can delete it, but this will be relaxed in the future when support for non-owner deletion is added to NAF.

The [`shooter.html` example shows an example](https://github.com/haydenjameslee/networked-aframe/blob/master/server/static/shooter.html#L79) where the bullets are deleted after 5 seconds of being spawned.


## That's all folks!

And there you have it. Your very own multi-user virtual reality experience running in a web browser!

Check out the Networked-Aframe documentation for more features and help. And again, be sure to post any questions on the [GitHub Issues page](https://github.com/haydenjameslee/networked-aframe/issues) or feel free to message me directly on [twitter](https://twitter.com/haydenlee37).

I would love love love you to send me cool examples you've made, and I'm looking to include more default examples with credit to the author, so [let me know](https://twitter.com/haydenlee37) what you make!

Cheers!

Hayden

[@HaydenLee37](https://twitter.com/haydenlee37)
