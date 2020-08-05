---
title: Road to Go pro - Part 1 - Installing Golang
date: 2020-08-01 00:00:00
featured_image: 'https://storage.googleapis.com/songxue-dev-images/content-images/2020-08-01-road-gopro-1/gopher1.jpg'
excerpt: 'Road to Go Pro is a series of short Go (the programming language) tutorials that take you from a beginner to a Pro. The first part is about installing Go and setting up the dev environment on your work station.'
---

## What is Go

Go is an open-source programming language developed by Google. It is a highly scalable and easy-to-learn language which suits for cloud-native development, serverless computing, edge computing and so on. Go is fairly young, comparing to vastly used Java and C#. But it has become a mainstream language in a very short time. Even one of the four biggest banks in Australia has adopted go in their technology stack.
I worked in projects which used Java/Kotlin and C# as the main programming languages before. Since last December, I started to write Go full time and so far I enjoyed it a lot. Although Go is not as feature-rich as the other two languages, it is still on the top of my list when it comes to developing microservices.

## Get things ready

Alright, let's start our road trip. The first thing we need to do is to get the development environment ready. Installing Go runtime in any operating system is painless. Let's get straight into it.

## Installation

### Mac OS

If you are using Mac OS, most likely you already have Homebrew installed. *Just in case you haven't, you can easily install it by following the instruction [here](https://brew.sh/).*

You can open your terminal and use the following command to install Go.

```bash
brew update && brew install golang
```

Next, you need to export three environment variables. You can add these into your terminal profile file (`.bash_profile` or `.zsh_profile` depending on which you use).

```bash
export GOROOT="$(brew --prefix golang)/libexec"
export GOPATH="${HOME}/go"
export PATH="$PATH:${GOPATH}/bin:${GOROOT}/bin"
```

Then you can reload it by running  
`source ~/.bash_profile` or `source ~/.zprofile`.

Well done so far! To verify the installation, you can run  

```bash
go version
```  

in the terminal. You should see the installed go version printed in the console.

### Windows

The easiest way to install Go on Windows is to use the MSI installer, which can be found [here](https://golang.org/doc/install) under Windows > MSI installer section. The installer should add necessary environment variables into `PATH` for you so you don't need to do that manually.  
*If you're using wsl (Windows Subsystem for Linux), please check out the Linux section below.*  
Easy right? Now to verify the installation, you can run  

```bash
go version
```  

in the command prompt or power shell. You should see the installed go version printed in the console.

### Linux

For Linux users, you need to download the archive from [here](https://golang.org/doc/install). It's a good habit to verify the checksum after downloading. If it looks all good, the next step is to extract it into `usr/local`. Similarly to Mac OS, you need to export the Go binary to the `PATH` environment variable.  

```bash
# Add this to your bash profile
export PATH=$PATH:/usr/local/go/bin
```

Once that's done, you can reload the profile using the `source` command. Finally, it's verification time. You can run  

```bash
go version
```  

in the terminal. You should see the installed go version printed in the console.

## IDE

<div class="gallery" data-columns="3">
    <img src="" style="display:none">
	<img src="https://storage.googleapis.com/songxue-dev-images/content-images/2020-08-01-road-gopro-1/road-to-go-pro-1-img-2.jpg">
    <img src="" style="display:none">
</div>

You can use any text editor for Go development. The mainstream IDEs are GoLand/IntelliJ (with Go plugin) from JetBrain and VS Code from Microsoft. I have used both of them and IntelliJ (with Go plugin) is my daily driver at the moment. It is a powerful IDE with rich features like quick refactoring, interface navigation, error detection, etc. And by the way, I'm using IntelliJ is because I can't get GoLand license from my employer. If getting a license is not a problem for you, I would recommend use GoLand as it is lighter than IntelliJ.  
Although JetBrains' product is nice to have, they come with a price. VS Code is a light-weight and versatile IDE. And most importantly, it's free and open-source. For this tutorial, we're going to use [VS Code](https://code.visualstudio.com/download).
There are some extensions you need to install before we can get started with coding. Here's the list:

1. [Go for VS Code](https://github.com/golang/vscode-go): provides rich language support for Go development in VS Code. This is an **essential** extension.
2. [Prettier](https://github.com/prettier/prettier-vscode): handy code formatter.

## Hello World

Okay, we are all set. Are you ready to build the famous hello world app?  
I think I heard yes. This is how it looks like in Go.

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello world!")
}
```

Pretty simple, right?

Let's open VS Code and create a file called `main.go`. Then copy-paste the code above into that file.  

Before we run this app, let's walk through each line and briefly explain what's going on there.

Line 1: declare the main package. Main package tells Go compiler that this package is an executable application.  
Line 3: import the `fmt` package that we will use later for print strings in the console.  
Line 5: declare the `main` function. Main function will be the entry point of the executable.  
Line 6: use the `fmt` package to print "Hello world!".  
Line 7: close main function.  

Enough talking, let's run your first Go app. Switch to the terminal window (shortcut in VS Code: control+`) and run the app using one of the following commands:  

```bash
go run .
# or build the app first and then run the binary
go build .
./helloworld
```

In the console, you should see `Hello world!`. Congratulations, everything works perfectly. You have successfully set up Go development environment and you are ready to start the journey to Pro.  

<div class="gallery" data-columns="3">
    <img src="" style="display:none">
	<img src="https://storage.googleapis.com/songxue-dev-images/content-images/2020-08-01-road-gopro-1/road-to-go-pro-1-img.jpg">
    <img src="" style="display:none">
</div>

In the next part, we will learn the basic syntax of Go. Stay tuned and I will see you in the next one.

**Please feel free to leave a comment below if you run into any problems or if you need a helping hand.**  
