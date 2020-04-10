---
title: Fight COVID-19 with homemade online version of an iconic board game - Power Grid
date: 2020-04-01 00:00:00
featured_image: 'https://storage.googleapis.com/songxue-dev-images/content-images/2020-04-01-virtualize-power-grid/featured.png'
excerpt: Our lives are more or less impacted by the COVID-19 pandemic by now. No doubt this is a terrible event for all of us and we want to get over it as soon as possible. However, it's a bit (very) hard to say goodbye to social life entirely and be happy to stay at home for the foreseeable future. So I spent some efforts to "bring" my friends together to play one of our favorite table-top board games, power grid. And of course, we are not physically together, please STAY AT HOME!
---

## Part 1

Australians had a rough start in 2020, a long and devastating bush fire took many precious things away from us, firemen, houses, wildlife, trees and so on. Not long after we finally got over the disaster, a pandemic hit us again and this time, we lost even more lives. COVID-19 has been spreading wildly across the globe and there's no sign of it slowing down at the moment. Most of the impacted countries have imposed some kind of rules to help stopping the spread. Australia is currently in stage 3 lock down, which means all of us should stay at home and avoid going out. No pubs, no friends gathering, no overseas travel. It's quite hard to get used to this new way of living. Before all of these, I longed for working from home because I have better setup at home than in the office. Whereas after two weeks of working from home, I kinda miss everything I had in the office, especially human interactions, badly.  

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

Enough of story-telling in part 1, now let's get to the details of the automated power plant market. I will firstly talk about the architecture of this application.

### Architecture

The architecture is very simple, what I need is an API to handle the interactions such as initiating the game, auction/buy power plants and etc and a simple UI for my friends to see which power plants are on the market. These are the two main components of this application. I chose Golang as the language for the backend, Angular as the frontend framework. The reason is simply because I have previous experience so I can use them efficiently. I don't want to waste any time on learning a new tech stack as I have a very tight deadline.

### Game server

I'm not going to explain how does the power plant market work here, if you're interested, you can find the rule book for Power Grid [here](http://riograndegames.com/getFile.php?id=2162).  

For the game server I developed, it's main job is to display a series of power plants and let users auction and buy them. The state of a game needs to be stored somewhere. The approach I chose is just use a global variable as the data store. The advantage of doing this is I can take out a component, database, from the architecture, thus this makes deployment a bit easier. The disadvantages or limitations are significant. For instance, I can only keep one game running at a time, this solution is pretty much impossible to scale and so on. I wouldn't recommend this approach if you're building a long-lived project but for a quick hack, it's perfect.  
This is how the database looks like this:  

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

_Note_: the concept of discounted power plants are introduced in the new 2019 recharged edition, just in case you're wondering what that is. Also, in the rule book, stage 3 is called step 3, I just find "stage" is a bit easier to understand.

With data store out of way, it's time to create the actual server. I'm using [Gin](https://github.com/gin-gonic/gin) as the web framework for the API server because it's very easy to use and its performance is fast. This is how the router looks like:

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

- Health endpoint is a standard endpoint to show whether the service is functioning.
- Powerplants endpoint returns all the power plant cards in the game, the response is static.
- Board endpoint returns the database value, aka the state of the game. The UI is going to consume this endpoint frequently.
- New game endpoint sets the database value based on the number of players and which country players are playing. I will dig into this part later.
- Auction endpoint alters a flag in the current market. This is not necessary but it improves the user experience a little bit. I will explain this later in the UI section.
- Buy endpoint triggers a replacement of power plant in the current market. It will be triggered by players.
- In Power Grid, a turn is consist of 5 phrases. Only phrase 2 and 5 (which is the end of turn) is related to the power plant market. Thus the end phrase and end turn endpoints trigger certain logic based on game rules.
- End game endpoint reset the values to default for the data store.
- CORS middleware is added because Angular enforce CORS by default.

#### New Game

Power Grid has lots of expansions, each expansion has two countries and some new rules adding on top of the basic rules. Most of the difference between expansions are the set up of the power plant deck. Some of the rules requires to remove cards, some requires to add cards, some even requires to order the deck. It's a good place to use interface to handle different implementations.  

```go
type CountryRule interface {
    InitDeck() []store.PowerPlant
    Shuffle(unadvancedDeck []store.PowerPlant, advancedDeck []store.PowerPlant) []store.PowerPlant
}
```

- `InitDeck` function handles the removal or addition of certain cards.
- `Shuffle` function handles the ordering of the deck. For example, China expansion orders the deck til plant number 30 something. So this logic is executed here.

This is the main components of the game server, it's very straight forward. Let's move onto the UI part now.

### Game UI

I'm not very good at writing UI so I kept it as simple as possible. To speed up the developing and still keep a decent look, I used ng-material library for all the UI components. They're predefined and looks pretty cool in my opinion.
There are only two pages for this application, namely starting page and market page.   

#### Start Page

<div class="gallery" data-columns="3">
	<img src="" style="display:none">
	<img src="https://storage.googleapis.com/songxue-dev-images/content-images/2020-04-01-virtualize-power-grid/UI-1.png">
	<img src="" style="display:none">
</div>

The starting page calls the new game endpoint to initialize a new game. As the server can't really handle multiple games running at the same time, I kindly asked all my friends to click the "Join game" button and leave the creation of a game to me. This is quite dangerous because if anyone at any time when we were playing clicked new game, it will overwrite the entire game state, which wipes all the fun. I'm glad that my friends listened to me and nothing like that happened when we played.

#### Market Page

<div class="gallery" data-columns="3">
    <img src="" style="display:none">
	<img src="https://storage.googleapis.com/songxue-dev-images/content-images/2020-04-01-virtualize-power-grid/UI-2.png">
    <img src="" style="display:none">
</div>

This is the page with most of the interactions. Players can start an auction and buy power plants. The auction button changes a flag to reveal the "Cancel" and "Buy" buttons. The reason I designed an extra step is I want to create a safety net for the buy power plant action. There's no turning back if a player clicked the wrong plant or they had a second thought. I didn't put in any roll back functionalities in the game server. This I believe decrease the chances of wrong operations to minimum and increased the usability of the application. Imagine if a player messed up a buy action, his experience of this game might be ruined. The last thing I want to see is users complaining about how dumb this application is although I know it's quite dumb.  
You might ask, how do you populates one's interaction to all players? The answer is brute force. I didn't have time to make this app a real-time application. The way I achieve this is ask the UI to query game state every second. And users shouldn't notice that the page is refreshing because Angular only refreshes the components that actually changed. This is a bad way of doing this kind of things because it increases the server load a lot and also the client requires for resources to do constant requests. Just like my decision to use in-memory data store, it has many disadvantages but Power Grid is a 2-6 players game, so at most I will get 360 request per second which my go server can easily handle. The only upside is this is the easiest solution to implement.

### Deployment

So my application is running smoothly on my local machine, it's time to deploy it to the cloud. I chose to use GCP as I already have a project set up on it. There are two options for easy deployment, I can use AppEngine to host them separately but that doubles the cost. Therefore, I chose the other option, which is a good old compute engine (VM). I don't need fancy specs for the VM and it's pretty cheap to run. The cost of AppEngine is about $0.12/hour and ComputeEngine is about $0.1/hour. So I created a ComputeEngine running Ubuntu LTS and cloned the two repositories. Running them is quite easy, however, there're a couple of annoying things I ran into. First of one, the lack of consistent environments. If for example, I want to spec up the VM, I will have to setup the environment again. Of course, this can be easily resolved by using Docker images but I didn't use it, why? Currently, two applications can run on 1 virtual CPU (half core) and 0.5G memory. Running docker will definitely require more resources than that. The other irritating thing is the IP address is not static. I don't plan to run the server 24/7, so every time I stop the VM, I need to change the IP address in code. Using static IP or keep it running can solve this problem but they're too expensive for this project. Also, I'm pretty sure there are more elegant ways to tackle this but I just didn't have the time to research.  
Now, everything is in place. I just need to post the url to my friends and then we can have an online power grid game together!

## Part 3

The gaming experience and what I have learned from this project.