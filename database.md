# TreeTap
## Database Artifact

### Narrative
Tree Tap was an app for Android that I had created a few years ago as a first-time collaboration with a new artist I had met from the r/INAT subreddit (prior to my ownership of the sub). We wanted to create a small game within the timeframe of a month or two that could be a proof of concept for us working together. Unfortunately, she ended up having some personal life issues arise and the game was never completed.  

Typically, free mobile games like this one would have multiple unlockable characters that the player could switch between. However, in lack of having those assets, perhaps the drive to play the game could be getting a top score on a leaderboard? Therefore for this artifact I decided to create a NoSQL database with a RESTful API that the app could use to store an online database of all players.  

I also used this as a chance to demonstrate my ability to create a RESTful API using industry standards like Javascript, Node, and the Express framework. As well show that I have experience with a NoSQL database like MongoDB. Essentially showcasing as a sort of “full-stack unity” developer who can do not only the front-end implementation of an API but also create one as well.  

The first challenge I faced was in my original intention of using MySQL on my namecheap vps, however they unfortunately have that system very locked down that would have made debugging a bit difficult. Instead, I transitioned to using MongoDB Atlas as they have a more straight-forward front end and allow me to make use of Google Cloud for my infrastructure in a more centralized data server location in the US.  

![mongodb](https://skytech6.github.io/SNHU-ePortfolio/images/treetap/mongodb.png)

My next problem was in the Unity side implementation. I have a good bit of experience with APIs from the code that I make for work being entirely reactive to server messages. However, this app never had UI created for a leaderboard. Instead, I recycled the continue menu & recolored some button UI to create a scalable top 5 high score list.  

![leaderboard](https://skytech6.github.io/SNHU-ePortfolio/images/treetap/leaderboard.png)

Overall, I’m pretty happy with the outcome.

### Android APK
[Universal APK](https://skytech6.github.io/SNHU-ePortfolio/downloads/treetap.apk) 

### Node.js Express
```javascript
const express = require("express");
const mongoClient = require("mongodb").MongoClient;
const objectId = require("mongodb").ObjectID;

const CONNECTION_URL = process.env.MONGODB_URI;
const DATABASE_NAME = "treetap";

var app = express();

app.use(express.json());
app.use(express.urlencoded({ extended: true }));

const port = process.env.PORT || 5000;

var database, collection;

app.listen(port, () => {
    mongoClient.connect(CONNECTION_URL, {useNewUrlParser: true, useUnifiedTopology: true},
        (error, client) => {
            if(error) {
                throw error;
            }

            database = client.db(DATABASE_NAME);
            require('./node_routes')(app, database);
            console.log("Connected to `" + DATABASE_NAME + "`!");
        });
});
```
### Node.js Express Routes
```javascript
var ObjectID = require('mongodb').ObjectID;

module.exports = function (app, db) {

    app.put('/leaderboard/:id', (req, res) => {
        const id = req.params.id;
        const details = { '_id': new ObjectID(id) };
        const newscore = { name: req.body.name, score: req.body.score };
        db.collection('leaderboard').update(details, newscore, (err, result) => {
            if (err) {
                res.send({ 'error': 'An error has occurred' });
            } else {
                res.send(newscore);
            }
        });
    });


    app.get('/leaderboard', (req, res) => {
        db.collection('leaderboard').find({}).toArray((err, result) => {
            if (err) {
                res.send({ 'error': 'An error has occurred' });
            } else {
                res.send(result);
            }
        });
    });

    app.get('/leaderboard/:id', (req, res) => {
        const id = req.params.id;
        const details = { '_id': new ObjectID(id) };

        db.collection('leaderboard').findOne(details, (err, item) => {
            if (err) {
                res.send({ 'error': 'An error has occurred' });
            } else {
                res.send(item);
            }
        });
    });


    app.post('/leaderboard', (req, res) => {
        const score = { name: req.body.name, score: req.body.score };
        db.collection('leaderboard').insertOne(score, (err, result) => {
            if (err) {
                res.send({ 'error': 'An error has occurred' });
            } else {
                res.send(result.ops[0]);
            }
        });
    });

    app.get('/top', (req, res) => {
        db.collection('leaderboard').find({}).sort({score: -1}).limit(5).toArray((err, result) =>{
            if(err) {
                res.send({ 'error': 'And error has occured'});
            } else {
                res.send(result);
            }
        });

    });
};
```
### Integration into Unity3D
```csharp
using System;
using System.Net;
using System.Net.Http;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Newtonsoft.Json;
using TMPro;

public class Leaderboard : MonoBehaviour
{
	public static Leaderboard instance;
    public static readonly HttpClient client = new HttpClient();
	string api = "https://treetap-highscores.herokuapp.com";
	public TMP_Text[] leaders;
	

    void Start()
    {
		if(instance == null)
		{
			instance = this;
		}
		else
		{
			Destroy(this);
		}

		GetLeaderboard();
    }

	async void GetLeaderboard()
	{
		try
		{
			var response = await client.GetStringAsync(api + "/top");
			var result = Newtonsoft.Json.JsonConvert.DeserializeObject<Entry[]>(response);
			

			for(int i = 0; i < 5; i++)
			{
				leaders[i].text = $"{result[i].name} - {result[i].score}";
			}
		}
		catch
		{
			Debug.Log("nope");
		}
	}

	public async void PostNewScore(int score)
	{
		Entry newEntry = new Entry
		{
			name = PlayerPrefs.GetString("Username"),
			score = score
		};

		var result = await client.PostAsJsonAsync(api + "/leaderboard", newEntry);
		var response = await result.Content.ReadAsStringAsync();
		Debug.Log(response);
	}

}

public class Entry
{
    public string name { get; set; }
	public int score { get; set; }
}
```

