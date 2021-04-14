# INAT Bot
## Software Design / Engineering Artifact

### Narrative
I originally created INAT Bot around 2 years ago to automate the moderation of a popular software/game development subreddit known as r/INAT (or I Need A Team). There are a few rules for making posts on INAT and checking each post that is made to the subreddit each day to make sure they were following the simple rules was quite the consumer of time. Enter my first Python bot to automate the process.  

I selected the bot as my design artifact because it was a weakly designed and hastily created script that simply “worked”, most the time. There was a bit of weird problems with error handling that would result in crashing, but with duct-taped multiprocessing to restart it which also did not work well. I thought it would be a good opportunity to showcase in a way my ability to take something from one language (Python) and convert it into another (JavaScript). Not to mention in 2021, it is probably a good idea to have a bit of example JS experience to show.  

I wanted the checks the bot made to the submissions to be cleaner; the structure of the reddit API wrapper I chose made this very easy to do with polling and the basic structure of methods in JavaScript. I did not want to deal with any multiprocessing jumbo; JavaScript is a single threaded asynchronous language! I wanted to cleanup the script of a lot of the clutter; I moved all the responses the bot makes to its own file and the credentials are now pulled from environmental variables.  

The objectives for this artifact were to have the project built in a way that it could foster collaborative development, having a well-thought-out design and principles, and fixing the vulnerabilities in the original script. I certainly think the objective covered the best was probably being able to have the bot apart of a more collaborative development. In fact, I have open-sourced the bot for the INAT community to submit bugs, pull requests, or just have a dialogue about the continued development about it. As well I do believe the design of this has been significantly improved as well a lot of vulnerabilities are no longer present with the usage of environmental variables and the removal of python’s messy multiprocessing library.  

The two dependencies I relied heavily on to create this bot in JavaScript are Snoowrap and Snoostorm which are respectively a wrapper for the reddit api in JS and an extension for said wrapper that includes polling. The previous version of the bot in Python also used polling with an older version of the PRAW wrapper; so, I decided I would stick with that design pattern for the new version.  

This is my first JavaScript script that serves an actual practical purpose. I had not come across “Promise” objects that are used for asynchronous web workers before, so even trying to read the documentation for Snoowrap/storm was a bit harder than I originally had expected it to be. As well there is a divide in JavaScript between CommonJS and ESM, but I just so happened to end up going with ESM which seems to be the future of Node… one day?

[Public GitHub](https://github.com/SkyTech6/INATBot/)
[See The Bot Live on r/INAT](https://www.reddit.com/r/INAT)
```javascript
import pkg from "snoostorm";
const { CommentStream, SubmissionStream } = pkg;
import Snoowrap from "snoowrap";
import getUrls from "get-urls";
import response from "./responses.js"; // Container for bot responses to users
import wordCounter from "./uniqueoccurances.js";

// Setup the Snoowrap client with variables passed from the Heroku env
const client = new Snoowrap({
    userAgent: 'INAT_BOT 0.2',
    clientId: process.env.CLIENT_ID,
    clientSecret: process.env.CLIENT_SECRET,
    username: process.env.REDDIT_USER,
    password: process.env.REDDIT_PASS
});

// The submission stream from snoostorm 
const submissions = new SubmissionStream(client, {
    subreddit: "inat",
    limit: 10, // pulls the latest 10 threads created
    pollTime: 20000, // does this check every 20 seconds, limited by Reddit's restrictions
});
submissions.on("item", submission => {
    // Check if the submission has already been moderated
    if (notModerated(submission)) {
        console.log(submission.title);
        let reply = "";

        // Checks if the post is a offer post
        if (includesWord("offer", submission.link_flair_text)) {
            // Offer posts require at least 150 words to be posted
            if (countWords(submission.selftext) < 150) {
                reply = response.offerLimit;
                submission.remove();
            }
        }
        else {
            // All other posts require at least 250 words to be posted
            if (countWords(submission.selftext) < 250) {
                reply = response.wordLimit;
                submission.remove();
            }
        }

        // Check if any urls exist in the submission
        if (getUrls(submission.selftext).size == 0) {
            reply = reply + response.url;
        }

        // Check if the word mmo appears in the body
        if (mmoCheck.test(submission.selftext.toLowerCase()) || mmoCheck.test(submission.title.toLowerCase())) {
            reply = reply + response.mmo;
        }

        // Check if any of the previous checks succeeded and added to the "reply" string
        if (reply != "") {
            submission.reply(reply);
        }

        // Checks for the percentage of unique word usage in the post
        if (uniquePercentage(submission.selftext) < 40) {
            client.composeMessage({
                text: submission.url,
                subject: "High Repetition Alert",
                to: "r/INAT"
            });
        }
    }
});

// The comment stream from snoostorm
const comments = new CommentStream(client, {
    subreddit: "inat",
    limit: 10, // Same limit and pollTime on both means 60 requests a minute / Reddit's max
    pollTime: 20000,
});
comments.on("item", comment => {
    let reply = "";

    // Check if the comment text contains the scope command
    if (includesWord("scope();", comment.body)) {
        reply = response.scope;
    }

    if (reply != "") {
        comment.reply(reply);
    }
});

// Splits a string by the spaces to get a total word count
const countWords = (str) => {
    return str.trim().split(/\s+/).length;
}

// Substring check
const includesWord = (word, str) => {
    return str.toLowerCase().includes(word);
}

// MMO Keyword check
const mmoCheck = new RegExp(
    ["mmo", "mmos", "mmorpg"].map(item => `\\b${item}\\b`).join("|")
)

const uniquePercentage = (str) => {
    let occurance = wordCounter(str, false);
    return (occurance.uniqueWordsCount * 100) / occurance.totalWords;
}

// Checks if the input submission already has a moderation post by the inat_bot
const notModerated = (submission) => {
    if (submission.num_comments > 0) {
        //snoowrap is all Promise-based, method takes place within the then callback
        client.getSubmission(submission.id).expandReplies().then(thread => {
            if (thread.comments.some(e => e['author']['name'] === "inat_bot")) {
                return false;
            }
            else {
                return true;
            }
        })
    }
    else {
        return true;
    }
}
```
