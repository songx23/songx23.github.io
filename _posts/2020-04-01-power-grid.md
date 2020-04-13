---
title: Fight COVID-19 with homemade online version of an iconic board game - Power Grid
date: 2020-04-01 00:00:00
featured_image: 'https://storage.googleapis.com/songxue-dev-images/content-images/2020-04-01-virtualize-power-grid/featured.png'
excerpt: Our lives are more or less impacted by the COVID-19 pandemic by now. No doubt this is a terrible event for all of us and we want to get over it as soon as possible. However, it's a bit (very) hard to say goodbye to social life entirely and be happy to stay at home for the foreseeable future. So I spent some efforts to "bring" my friends together to play one of our favorite table-top board games, power grid. And of course, we are not physically together, please STAY AT HOME!
---

## Part 1

Australians had a rough start in 2020, a long and devastating bush fire took many precious things away from us, firemen, houses, wildlife, trees and the list goes on. Not long after we finally got over the disaster, a pandemic hit us again and this time, we lost even more lives. COVID-19 has been spreading wildly across the globe and there's no sign of it slowing down at the moment. Most of the impacted countries have imposed some kind of rules to help stopping the spread. Australia is currently in stage 3 lock down, which means all of us should stay at home and avoid going out. No pubs, no friends gathering, no overseas travel. It's quite hard to get used to this new way of living. Before all of these, I longed for working from home because I have better setup at home than in the office. Whereas after two weeks of working from home, I kinda miss everything I had in the office, especially human interactions, badly.  

One day, I was chatting with my partner about how I miss the times we sat together with friends and play Power Grid and how unfortunate this game doesn't have an online version. All of a sudden, she said "why don't we move this game online?" I laughed and replied "I'm not a game developer, plus how much effort do you think it will take to virtualize such a complicated game? Even after playing it three times, we still need to go back to the rule book." She dragged me up from the sofa and said confidently "Not necessary, it can be quite easy, in fact, I have a perfect tool to do this, come check this out." I was reluctant to move my body but I gave in and walked to our study with her.  

She opened her laptop and went to a website called "Figma". I asked "I've heard you talking about Figma, what is it exactly?"  
"I use it everyday for work. It's like a drawing pad but you can work on it collaboratively with others in real time. Help me take out the game board of Power Grid."  
I was still not quite sure what she's up to but I took out the game board as she asked. She quickly shot a photo of the game board and uploaded to Figma. Then she started to drag icons representing game elements such as resources (coal, oil, etc), cities and power plants onto it. After about 10 to 15 minutes, she asked me to open up a link from my laptop. What in front of me is a mock up of the game, I can see cities tokens, resource tokens placed on the game map, looked just like it's a photo of the game.  
<div class="gallery" data-columns="1">
	<img src="https://storage.googleapis.com/songxue-dev-images/content-images/2020-04-01-virtualize-power-grid/featured.png">
</div>
"You can drag your tokens on or off the game board easily like this. We can start a group voice chat and play Power Grid on Figma just like in real life." She started to explain.  
"Very interesting! Moving tokens totally works. However, how can we represent the power plant market on Figma? We can't shuffle cards on it, can we?" I replied.  
She shook her head, "No, I don't know how to program that kind of stuff. That's what your job."  
I shrugged, "How am I suppose to know how to program stuff on Figma, I've never used it before... Hold on, we don't have to use Figma, I think I can write an app to represent the power plant market." Lots of ideas flew through my head. Fundamentally, the power plant market is an array of numbers ordered and shown in certain ways to players. It shouldn't be a hard program to code.  
"Challenge accepted!" I said excitedly. "I will write an app for the power plant market and you can work on the game board, adding all the elements."  
She smiled and said "Sounds like a plan, can you finish your app by the end of next week? I want to play this with friends on Saturday."  
"No problem, I can hack something up quickly. It's not gonna look good by the way, I suck at UI stuff, you have to accept that."  
"I will take whatever you build as long as it works." said she.
It was a Saturday evening and we were excited about this little project.

## Part 2

Alright, enough of story-telling, now let's get to the details of this automated power plant market I built. Firstly, let's talk about the architecture of this application.

### Architecture

To develop this application in a short time frame, I want to keep the architecture simple, the less moving components the better. What I came up with is an API (game server) to handle the logic such as initiating the game, auction/buy power plants and etc, a simple UI for my friends to see which power plants are on the market and interact with the game server. These are the two main components of this application. I chose Golang for API development and Angular as the frontend framework simply because I have previous experience with them. I don't want to waste any time on learning a new tech stack for this particular project.

### Game server

The game server is the brain of all these, it enforces the logic that reflects the rules. I'm not going to explain how does the power plant market work here, if you're interested, you can find the rule book for Power Grid [here](http://riograndegames.com/getFile.php?id=2162).  

The main job of game server is to maintain the state of power plant market, which needs to be stored somewhere. As you might noticed already, I didn't include a database of some sort in the architecture. The approach I chose is to just use a global variable as the data store. The advantage of doing this is I can be lazy and not code the database layer. The disadvantages or limitations are significant. For instance, I can only keep one game running at a time, it is pretty much impossible to scale and so on. I wouldn't recommend this approach if you're working on a long-lived project but for a quick hack, it's acceptable.  
This is how the data structure looks like:  

```go
type Database struct {
    CurrentMarket     []PowerPlant
    CurrentDeck       []PowerPlant
    DiscountPlantSold bool
    CurrentTurn       int
    ReachedStageThree bool
    Logs              []string
}

type PowerPlant struct {
    ID           int          `json:"id"`
    Resource     ResourceType `json:"resource"`
    ResourceAmt  int          `json:"resourceAmount"`
    PoweringCap  int          `json:"poweringCapability"`
    IsAdvanced   bool         `json:"isAdvanced"`
    IsDiscounted bool         `json:"isDiscounted"`
    IsBidding    bool         `json:"isBidding"`
}
```

_Note_: the concept of discounted power plants is introduced in the new 2019 recharged edition, just in case you're wondering what that is. Also, in the rule book, stage 3 is called step 3, I just find "stage" is a bit easier to understand.

With this out of way, it's time to create the actual server. I'm using [Gin](https://github.com/gin-gonic/gin) as the web framework because it's very easy to use and its performance is fast. This is how the router looks like:

```go
func (r *RestHandler) createRoutes() {
    api := r.Router.Group("/api")
    {
        api.GET("/health", r.healthCheckHandler)
        api.GET("/powerplants", r.getAllPowerPlantsHandler)
        api.GET("/board", r.getCurrentBoardHandler)
        api.POST("/newgame", r.startNewGameHandler)
        api.PUT("/auction", r.auctionHandler)
        api.PUT("/buy", r.buyHandler)
        api.PUT("/endphrase", r.endPhraseHandler)
        api.PUT("/endturn", r.endTurnHandler)
        api.DELETE("/endgame", r.endGameHandler)
    }

    r.Router.Use(cors.Default())
}
```

The following list shows what function does each endpoint fulfill:

- Health endpoint is a standard endpoint to show whether the service is running.
- Powerplants endpoint returns all the power plant cards in the game, the response is static.
- Board endpoint returns the game state, aka the global database variable. UI is going to consume this endpoint frequently.
- New game endpoint sets the game state based on the number of players and which country is being played.
- Auction endpoint alters a flag in the current market. This is not necessary but it improves the user experience. I will explain this later in the UI section.
- Buy endpoint replaces a power plant in the current market based on current game stage.
- In Power Grid, a turn is consist of 5 phrases. The end phrase and end turn endpoints trigger certain rules when activated by ending of phrase 2 and 5.
- End game endpoint resets the game state to default values.

I'm using CORS middleware because Angular enforce CORS by default.

#### New Game

I want to talk a bit more about what happens when creating a new game. Power Grid is a quite old board game so it has lots of expansions, each expansion provides a new game board showing two different countries. Some new rules are also added to provide new gaming experience. Most of the differences between expansions (_*I have only checked about 4 expansions so it doesn't cover all_) lie in the set up of the power plant deck. Some expansions require to remove power plant cards from deck, some requires to add cards, some even requires to order the deck. This represents a good place to use interface to handle different implementations. Here's the interface I defined.   

```go
type CountryRule interface {
    InitDeck() []store.PowerPlant
    Shuffle(unadvancedDeck []store.PowerPlant, advancedDeck []store.PowerPlant) []store.PowerPlant
}
```

- `InitDeck` function handles the removal or addition of certain cards.
- `Shuffle` function handles the ordering of the deck. For example, China expansion orders the deck til plant number 30. So this logic is executed here.

These are the main components of the game server, the structure is pretty straightforward. The implementation is just a bunch manipulations of two arrays.  
Let's move on to the UI part now.

### Game UI

I'm not an UI expert so I kept everything as simple as possible. Besides, I'm a lazy person, I prefer to use existing library to speed things up. I picked ng-material library for UI components as they look pretty cool and Material is natively supported by Angular.  
There are only two pages for this application, namely starting page and market page.  

#### Start Page

<div class="gallery" data-columns="3">
	<img src="" style="display:none">
	<img src="https://storage.googleapis.com/songxue-dev-images/content-images/2020-04-01-virtualize-power-grid/UI-1.png">
	<img src="" style="display:none">
</div>

The starting page is a simple form that collects essential play information. The "Start new game" button calls the new game endpoint to initialize a new game. As the server can't really handle multiple games running at the same time, I kindly asked all my friends to click the "Join game" button and leave the creation to me. This is quite dangerous because if anyone at any time clicked the start game button, it will overwrite the entire game state and there's no way to recover the previous state, hence using a single global variable as database is a really bad idea. Fortunately, nothing like that happened when we played.

#### Market Page

<div class="gallery" data-columns="3">
    <img src="" style="display:none">
	<img src="https://storage.googleapis.com/songxue-dev-images/content-images/2020-04-01-virtualize-power-grid/UI-2.png">
    <img src="" style="display:none">
</div>

This is the page which players can actually interact "freely". Anyone can start an auction and buy a power plant through this page. The auction button changes a flag in game state so that it can reveal the "Cancel" and "Buy" buttons. The reason I designed an extra step is for creating a safety net when buying power plants. There's no turning back if a player clicked the wrong plant or they had a second thought because there's no roll back functionalities in the game server. This is disastrous for a player when mis-clicked a button. I don't want to punish my friends like this. Therefore, this feature is quite necessary.  
You might ask, how do you populate one's interaction to all players? I didn't have time to make this a real-time application in the end so the answer is using brute force, make UI queries game state every second. This is another bad decision I know but it works. Users shouldn't really notice that the page is constantly refreshing thanks to Angular's refreshing mechanism and speed. At a glance, this approach increases the server load and also requires more client resources. Whereas Power Grid is a 2-6 players game, so at most I will get 360 request per second which the API server can easily handle.  

### Deployment

So the application is running smoothly on my local machine, it's time to deploy it to the cloud. I chose to use GCP as I already have a project set up on it. There are two options for quick deployment, I can use AppEngine to host the UI and API separately, or I can use a good old VM (ComputeEngine). The cost of two AppEngine is about $0.12/hour and ComputeEngine (low spec, shared resource) is about $0.1/hour. Obviously, I went for the cheapest option.  
Running them on Ubuntu LTS is quite easy. However, there're a couple of annoying things I want to point out.

1. The lack of Docker usage. I will have to setup the whole environment again if I want to spec up or create a VM. Currently, two applications can run on 1 virtual CPU (half physical core) and 0.5G memory. Running docker will definitely require more resources than that so I chose not to use it for now.
2. Non-static IP address. I don't plan to run the server 24/7, so every time I stop the VM, I need to change the IP address in environment. On top of my head, using static IP or keeping the VM running can solve this problem but they're too expensive for this project.
Now, everything is in place. I just need to post the url to my friends and then we can have an online power grid game together!

## Part 3

It's finally time to put everything into test. On a Saturday, my partner and I kicked off a group video chat and introduced the system we built. They were amazed how we put everything together in just a week. We played for 3 hours that day and surprisingly, our system didn't break down, everything works just "fine" but we were exhausted after it.  
It feels totally different to play table top game online in a bad way. Often time, we all spoke at the same time and then we need to repeat ourselves again so that we can understand what others are saying. On the other hand, moving virtual tokens on the board is quite hard and slow, I found lots of time when we were waiting for others to finish their actions, our attention was distracted to other places like YouTube. Last but not the least, most of us were using laptop, it's quite hard to see the whole board without zooming in and out. I missed lots of things outside of my sight and that led to some bad choices. All of above resulted, to be honest, a quite unpleasant gaming experience.  
Later on, I found what we built is called tabletop simulator. There're some existing software doing the exact same thing (none of them have PowerGrid though). My partner and I tried a couple simulators afterwards and had the same feelings. I guess it's really hard to migrate table top board game online while maintaining the same gaming experience.  
I'm glad that I didn't spend too much time on this project as it's very likely that I'm going to park it for a while, although the building process was really fun. It's been a while since the last time I participate in a hackathon. Good to find that feeling back.  

## The most important thing

PLEASE stay at home if you can to help fight COVID-19. The sooner we can take this under control, the sooner we can get our pre-COVID19 lifestyle back. I don't know about you but I really miss it.
