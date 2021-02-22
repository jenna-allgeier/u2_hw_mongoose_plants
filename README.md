# Mongoose Plants API

![barnsely's](https://i.ytimg.com/vi/iGMGVpLMtMs/maxresdefault.jpg)

## Overview
In this deliverable we'll be building and deploying our very own custom API about plants! We'll be building it from the ground up to cover the entire process using `express.Router`.

## Getting started

- `Fork` and `clone` this ropository

## Express, Mongoose, & MongoDB

Let's start!

```sh
cd mongodb-mongoose-express-using-router
npm init -y && npm install mongoose
mkdir db models seed
touch db/index.js models/plant.js seed/plants.js
```

Now let's open up Visual Studio Code and write some code:

```sh
code .
```

Inside our `db` folder we are going to use Mongoose to establish a connection to our MongoDB `plantsDatabase`:

mongodb-mongoose-express-using-router/db/index.js
```js
const mongoose = require('mongoose')

let MONGODB_URI = process.env.PROD_MONGODB || process.env.MONGODB_URI || 'mongodb://127.0.0.1:27017/plantsDatabase'

mongoose
    .connect(MONGODB_URI, { useUnifiedTopology: true, useNewUrlParser: true })
    .then(() => {
        console.log('Successfully connected to MongoDB.')
    })
    .catch(e => {
        console.error('Connection error', e.message)
    })

const db = mongoose.connection

module.exports = db
```

Although, MongoDB is schema-less, Mongoose allows us to write a schema for our plant model which makes it nice to know what is a plant in our database and what a plant "looks" like in our database:

mongodb-mongoose-express-using-router/models/plant.js
```js
const mongoose = require('mongoose')
const Schema = mongoose.Schema

const Plant = new Schema(
    {
        name: { type: String, required: true },
        description: { type: String, required: true },
        image: { type: String, required: true },
    },
    { timestamps: true },
)

module.exports = mongoose.model('plants', Plant)
```

Cool. We have a "blueprint" for what a plant is. Let's now use it and create plants.

mongodb-mongoose-express-using-router/seed/plants.js
```js
const db = require('../db')
const Plant = require('../models/plant')

// Connect to the database
db.on('error', console.error.bind(console, 'MongoDB connection error:'))

const main = async () => {
    const plants = [
        { name: 'Aloe Vera', description: 'Aloe vera is a succulent plant species of the genus Aloe. An evergreen perennial, it originates from the Arabian Peninsula, but grows wild in tropical, semi-tropical, and arid climates around the world. It is cultivated for agricultural and medicinal uses.', image: 'https://upload.wikimedia.org/wikipedia/commons/4/4b/Aloe_vera_flower_inset.png' },
        { name: 'Snake Plant', description: 'Sansevieria trifasciata is a species of flowering plant in the family Asparagaceae, native to tropical West Africa from Nigeria east to the Congo. It is most commonly known as the snake plant, Saint Georges sword, mother-in-laws tongue, and vipers bowstring hemp, among other names.', image: 'https://upload.wikimedia.org/wikipedia/commons/thumb/f/fb/Snake_Plant_%28Sansevieria_trifasciata_%27Laurentii%27%29.jpg/2560px-Snake_Plant_%28Sansevieria_trifasciata_%27Laurentii%27%29.jpg' },
        { name: 'Areca palm', description: 'Dypsis lutescens, also known as golden cane palm, areca palm, yellow palm, or butterfly palm, is a species of flowering plant in the family Arecaceae, native to Madagascar and naturalized in the Andaman...', image: 'https://upload.wikimedia.org/wikipedia/commons/thumb/4/4b/Dypsis_lutescens1.jpg/1280px-Dypsis_lutescens1.jpg' },
        { name: 'Spider Plant', description: 'Chlorophytum comosum, often called spider plant but also known as airplane plant, St. Bernards lily, spider ivy, ribbon plant, and hen and chickens is a species of perennial flowering plant. It is native to tropical and southern Africa, but has become naturalized in other parts of the world, including western Australia.', image: 'https://upload.wikimedia.org/wikipedia/commons/thumb/b/b1/Hierbabuena_0611_Revised.jpg/1920px-Hierbabuena_0611_Revised.jpg' },
        { name: 'Dracaena', description: 'Dracaena is a genus of about 120 species of trees and succulent shrubs. In the APG IV classification system, it is placed in the family Asparagaceae, subfamily Nolinoideae (formerly the family Ruscaceae). It has also formerly been separated (sometimes with Cordyline) into the family Dracaenaceae or placed in the Agavaceae (now Agavoideae).', image: 'https://upload.wikimedia.org/wikipedia/commons/f/f7/Dracaena_draco.jpg' },
        { name: 'Weeping Fig', description: 'Ficus benjamina, commonly known as weeping fig, benjamin fig or ficus tree, and often sold in stores as just ficus, is a species of flowering plant in the family Moraceae, native to Asia and Australia. It is the official tree of Bangkok.', image: 'https://upload.wikimedia.org/wikipedia/commons/thumb/9/9d/Ficus_benjamina2.jpg/1280px-Ficus_benjamina2.jpg' },
        { name: 'Peace Lily', description: 'Spathiphyllum is a genus of about 40 species of monocotyledonous flowering plants in the family Araceae, native to tropical regions of the Americas and southeastern Asia. Certain species of Spathiphyllum are commonly known as spath or peace lilies.', image: 'https://upload.wikimedia.org/wikipedia/commons/thumb/b/bd/Spathiphyllum_cochlearispathum_RTBG.jpg/1024px-Spathiphyllum_cochlearispathum_RTBG.jpg' }
    ]

    await Plant.insertMany(plants)
    console.log("Created some plants!")
}
const run = async () => {
    await main()
    db.close()
}

run()
```

Awesome, so this plants "seed" file above is a script that, once executed, connects to the Mongo database and creates 7 plants in the plants collection.

Let's execute our plants seed file:

```sh
node seed/plants.js
```

So how do we know if it worked? We could drop into the `mongo` interactive shell and check:

```sh
mongo
> use plantsDatabase
> db.plants.find()
```

Create a .gitignore file `touch .gitignore`!

```sh
/node_modules
.DS_Store
```

Cool, enough Mongoose. Now, Express. Let's install Express and Nodemon for development:

```sh
npm install express
npm install nodemon -D
```
And now let's setup our express folders:

```sh
mkdir routes controllers
touch server.js routes/index.js controllers/index.js
```

Modify your package.json file:

```js
....
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "nodemon server.js"
  },
....
```

Let's setup the root route:

mongodb-mongoose-express-using-router/routes/index.js
```js
const { Router } = require('express');
const router = Router();

router.get('/', (req, res) => res.send('This is root!'))

module.exports = router;
```

Inside of server.js:
```js
const express = require('express');
const routes = require('./routes');
const db = require('./db');

const PORT = process.env.PORT || 3000;

const app = express();

app.use('/api', routes);

db.on('error', console.error.bind(console, 'MongoDB connection error:'))

app.listen(PORT, () => console.log(`Listening on port: ${PORT}`))
```

Test the route:
```sh
npm start
```

Test the root endpoint in your browser: http://localhost:3000/api/

Good, now let's work on the controller. The controller is where we will set up all of our logic e.g. what does the API do when we want to create a new plant? Update a plant? etc.

mongodb-mongoose-express-using-router/controllers/index.js
```js
const Plant = require('../models/plant');

const createPlant = async (req, res) => {
    try {
        const plant = await new Plant(req.body)
        await plant.save()
        return res.status(201).json({
            plant,
        });
    } catch (error) {
        return res.status(500).json({ error: error.message })
    }
}

module.exports = {
    createPlant,
}
```

Remember we will need the express body-parser middleware to access the req.body object so:

```sh
npm i body-parser
```

Run the server again:

```sh
npm start
```

Add the following lines of code to the top of server.js:

```js
const bodyParser = require('body-parser');
app.use(bodyParser.json())
```

Cool. We have the logic to create a new plant. Now let's create a route on our server to connect the request with the controller:

mongodb-mongoose-express-using-router/routes/index.js:
```js
const { Router } = require('express');
const controllers = require('../controllers')
const router = Router();

router.get('/', (req, res) => res.send('This is root!'))

router.post('/plants', controllers.createPlant)

module.exports = router;
```

Make sure your json api server is running:
```sh
npm start
```

Use Postman (POST) method to test the create route (http://localhost:3000/api/plants):

```js
{
    "name": "Test Plant",
    "description": "Test Description",
    "image": "https://testimage.com/plant.png"
}
```

Awesome! Now I want to create a controller method to grab all the plants from the database:

mongodb-mongoose-express-using-router/controllers/index.js
```js
const Plant = require('../models/plant');

const createPlant = async (req, res) => {
    try {
        const plant = await new Plant(req.body)
        await plant.save()
        return res.status(201).json({
            plant,
        });
    } catch (error) {
        return res.status(500).json({ error: error.message })
    }
}

const getAllPlants = async (req, res) => {
    try {
        const plants = await Plant.find()
        return res.status(200).json({ plants })
    } catch (error) {
        return res.status(500).send(error.message);
    }
}

module.exports = {
    createPlant,
    getAllPlants
}
```

Add the following route to your ./routes/index.js file:
```js
router.get('/plants', controllers.getAllPlants)
```

Open http://localhost:3000/api/plants in your browser or do a GET request in Postman.

Nice, now let's add the ability to find a specific plant:

mongodb-mongoose-express-using-router/controllers/index.js
```js
const getPlantById = async (req, res) => {
    try {
        const { id } = req.params;
        const plant = await Plant.findById(id)
        if (plant) {
            return res.status(200).json({ plant });
        }
        return res.status(404).send('Plant with the specified ID does not exists');
    } catch (error) {
        return res.status(500).send(error.message);
    }
}
```

Add it to the export:

mongodb-mongoose-express-using-router/controllers/index.js
```js
module.exports = {
    createPlant,
    getAllPlants,
    getPlantById
}
```

Add the route:

mongodb-mongoose-express-using-router/routes/index.js
```js
router.get('/plants/:id', controllers.getPlantById)
```

Test it! http://localhost:3000/api/plants/5e38921e9c3bd077f50dc9a2

This is a good point to integrate better logging. Right now, if we check our terminal when we hit the http://localhost:3000/api/plants/5e38921e9c3bd077f50dc9a2 endpoint we see the raw SQL that was executed. For debugging purposes and overall better logging we're going to use an express middleware called morgan:

```sh
npm install morgan
```

Add the following to your server.js file:
```js
const logger = require('morgan');
app.use(logger('dev'))
```

Let's see the result:
```sh
npm start
open http://localhost:3000/api/plants/5e38921e9c3bd077f50dc9a2
```

You should now see in your terminal something like this:
```sh
GET /api/plants/5e38921e9c3bd077f50dc9a2 200 14.273 ms
```

That's morgan!

So we can now create plants, show all plants, and show a specific plant. How about updating a plant and deleting a plant?

mongodb-mongoose-express-using-router/controllers/index.js
```js
const updatePlant = async (req, res) => {
    try {
        const { id } = req.params;
        await Plant.findByIdAndUpdate(id, req.body, { new: true }, (err, plant) => {
            if (err) {
                res.status(500).send(err);
            }
            if (!plant) {
                res.status(500).send('Plant not found!');
            }
            return res.status(200).json(plant);
        })
    } catch (error) {
        return res.status(500).send(error.message);
    }
}

const deletePlant = async (req, res) => {
    try {
        const { id } = req.params;
        const deleted = await Plant.findByIdAndDelete(id)
        if (deleted) {
            return res.status(200).send("Plant deleted");
        }
        throw new Error("Plant not found");
    } catch (error) {
        return res.status(500).send(error.message);
    }
}
```

Make sure your exports are updated:
```js
module.exports = {
    createUser,
    getAllUsers,
    getUserById,
    updatePlant,
    deletePlant
}
```

Let's add our routes:

mongodb-mongoose-express-using-router/routes/index.js
```js
router.put('/plants/:id', controllers.updatePlant)
router.delete('/plants/:id', controllers.deletePlant)
```

Test update (PUT) in Postman. Your request body in Postman will have to look something like this:

http://localhost:3000/api/plants/5e38921e9c3bd077f50dc9a2

```js
{
    "name": "Update Plant Test",
    "description": "Test Description",
    "image": "https://testimage.com/plant.png"
}
```

Test delete (DEL) in Postman using a URL like this http://localhost:3000/api/plants/5e38921e9c3bd077f50dc9a2

Success! We built a full CRUD JSON API in MongoDB, Mongoose, and Express using Express Router!

## Deployment
![](https://miro.medium.com/max/1320/1*owg5RPtazedwH8fxpZF_vg.png)
> Image from [heroku.com](https://www.heroku.com)

Let's deploy our app to [heroku](https://devcenter.heroku.com/articles/heroku-cli#download-and-install).

First we need to update our package.json:

```js
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node server.js",
    "dev": "nodemon server.js"
  },
```

> Make sure you're on the `master` branch!

1. `heroku create your-heroku-app-name`
2. `heroku buildpacks:set heroku/nodejs`
3. `heroku addons:add mongolab`
4. `git status`
5. `git commit -am "add any pending changes"`
6. `git push heroku master`
7. `heroku run node seed/plants.js`

> Having issues? Debug with the Heroku command `heroku logs --tail` to see what's happening on the Heroku server.

Test the endpoints :)

> https://your-heroku-app-name.herokuapp.com/api/plants

## Bonus: Advanced Deployment using MongoDB Atlas

In the previous deployment we deployed to a Heroku. Heroku created a sandbox MongoDB database which is suitable for testing and development and small production apps. For larger apps that you plan on scaling, MongoDB recommends you use MongoDB Atlas. In this next step we are going to learn how to deploy our app to Heroku and create a production-ready MongoDB database using MongoDB Atlas.

[MongoDB Atlas](https://www.mongodb.com/cloud/atlas) manages our database beautifully - things like scaling the database as it grows, backups, and many more features are all handled by [MongoDB Atlas](https://www.mongodb.com/cloud/atlas).

First step is to signup to [MongoDB Atlas](https://www.mongodb.com/cloud/atlas) here: https://www.mongodb.com/cloud/atlas

> Then click the green button "Build a Cluster"

<p align="center">
  <img src="https://i.imgur.com/KcgbFXc.png" width="80%"/>
</p>

##

> Select the **free** tier

<p align="center">
  <img src="https://i.imgur.com/vAgMBoM.png" width="60%"/>
</p>

##

> Select AWS, the N. Virginia free tier, and click the green "Create Cluster" button

<p align="center">
  <img src="https://i.imgur.com/ylkZwtl.png" width="60%"/>
</p>

##

> Grab a cup of coffee ☕ it will take a few minutes for MongoDB Atlas to create your databases in the cloud

<p align="center">
  <img src="https://i.imgur.com/yVn9eLy.png" width="80%"/>
</p>

##

> Click the "CONNECT" button

<p align="center">
  <img src="https://i.imgur.com/OfroaVP.png" width="80%"/>
</p>

##

> You should see a modal pop up. Click "Add a Different IP Address" button

> Then enter 0.0.0.0/0 for the IP Address input (this will allow all IP Addresses to access the database)

> Click the green "Add IP Address" button

> Now let's secure the database. Create a username/password (remember the password, you will need it later)

> Click the "Create MongoDB User" button

> Next, click the "Choose a connection method" button in the lower right of the modal

<p align="center">
  <img src="https://i.imgur.com/hv860VE.png" width="60%"/>
</p>

##

> Click "Connect Your Application"

<p align="center">
  <img src="https://i.imgur.com/40KJqQt.png" width="60%"/>
</p>

##

> Select the "Connection String Only" tab

> Copy the connection string

<p align="center">
  <img src="https://i.imgur.com/vZnBMKN.png" width="60%"/>
</p>

##

Now we're ready to deploy to Heroku and specify our MongoDB Atlas URI connection string to tell Heroku where our database lives.

> Make sure you're on the `master` branch!

1. `heroku create your-new-heroku-app-name`
> Note: If you already deployed using the previous deployment lesson you will need to change your heroku remote url:

```sh
git remote set-url heroku https://git.heroku.com/your-new-heroku-app-name.git
```

2. `heroku buildpacks:set heroku/nodejs`
3. `heroku config:set PROD_MONGODB="<INSERT YOUR MONGODB URI CONNECTION STRING HERE>"`
    - replace the word test in your connection string with the name of your a database: plantsDatabase
4. `git status`
5. `git commit -am "add any pending changes"`
6. `git push heroku master`
7. `heroku run node seed/plants.js`

Test the endpoints :)

> https://your-heroku-app-name.herokuapp.com/api/plants

**Excellent!**

![](http://www.winsold.com/sites/all/modules/winsold/images/checkmark.svg)


## Resources

- https://devcenter.heroku.com/articles/mongolab
