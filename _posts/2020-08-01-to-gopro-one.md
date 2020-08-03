---
title: Road to Go pro - Part 1 - Installing Golang
date: 2020-07-30 00:00:00
featured_image: 'https://storage.googleapis.com/songxue-dev-images/content-images/2020-08-01-road-gopro-1/gopher1.jpg'
excerpt: 'Road to Go pro is a series of short Go (not the game) tutorials that takes you from a beginner to a pro. This first part is about installing Go and setting up the dev environment on your work station.'
---

## What is Go

Go is an open source programming language developed by Google. Go is a highly scalable and easy-to-learn language that is suitable for cloud-native development, serverless computing, IoT and so on. Go is fairly young, comparing to vastly used Java and C#. But it has become a mainstream language in a very short time. Even for one of the four biggest banks in Australia has adopted go in their technology stack.
I have worked in projects which use Java/Kotlin and C# as the main programming languages before. Since last December, I started to write Go full time and so far I enjoyed it. Although Go is not as feature rich as the other two mainstream languages, it is still on the top of my list when it comes to develop microservices.

## Get things ready

Alright, let's start our road trip. The first thing we need to do is to get the development environment ready. It is very easy to install Go runtime in any operating system. Let's get into it.

## Installation

### Mac OS

If you are using Mac OS, most likely you already have Homebrew installed. If not, you can easily install it by following the instruction [here](https://brew.sh/).

Now, you can open your terminal and use the following command to install Go.

```bash
brew update && brew install golang
```

Now you need to export three environment variables. You can add these into your terminal profile file (either `.bash_profile` or `.zsh_profile`).

```bash
# Add these to your profile file
export GOROOT="$(brew --prefix golang)/libexec"
export GOPATH="${HOME}/go"
export PATH="$PATH:${GOPATH}/bin:${GOROOT}/bin"
```

Once these `export`s are in your profile, you can reload it by running  
`source ~/.bash_profile` or `source ~/.zprofile`.

Well done so far! To verify the installation, you can run  

```bash
go version
```  

in the terminal. You should see the installed go version printed in the console.

### Windows

The easiest way to install Go on Windows is to use the MSI installer, which can be found [here](https://golang.org/doc/install) under Windows > MSI installer section. The installer should add the necessary environment variable into `PATH` for you. If you're using wsl (Windows Subsystem for Linux), please checkout the Linux section below.  
Easy right? Now to verify the installation, you can run  

```bash
go version
```  

in the terminal. You should see the installed go version printed in the console.

### Linux

For Linux users, you need to down the archive from [here](https://golang.org/doc/install). It's a good habit to verify the checksum after downloading. If it looks all good, the next step is to extract it into `usr/local`. Similarly to Mac OS, you need to export the Go binary to the `PATH` environment variable.  

```bash
# Add this to your bash profile
export PATH=$PATH:/usr/local/go/bin
```

Once the `PATH` environment variable is exported, you could reload the profile using `source` command. Finally, it's verification time. You can run  

```bash
go version
```  

in the terminal. You should see the installed go version printed in the console.

## IDE

I have used IntelliJ Ultimate (with Go plugin, equivalent with GoLand) and VS Code for Go development. IntelliJ Ultimate and GoLand are powerful IDEs packed with rich features but they are not free products. VS Code is a light-weight and versatile IDE and it is just as powerful as Jetbrain's product. And most importantly, it's free and open-source. For the purpose of this tutorial, we're going to use [VS Code](https://code.visualstudio.com/download).
There are some extensions you need to install before we can get started with coding. Here's the list:

1. [Go for VS Code](https://github.com/golang/vscode-go): provides rich language support for Go development in VS Code. This is an **essential** extension.
2. [Prettier](https://github.com/prettier/prettier-vscode): handy code formatter.
3. [Docker for VS Code](https://github.com/microsoft/vscode-docker): provides support for building, managing and deploying docker containers. This is an *optional* extension.

## Hello World

We are all set. Are you ready to build the famous hello world application?  
Alright, open your VS Code and create a file called `main.go`.  
Key in the following lines to `main.go`.  

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello world!")
}
```

This is the famous helloworld app, pretty simple, right?  
Let me walk you through each line and explain what's happening here.

Line 1: define the main package.  
Line 3: import the `fmt` package that we will use later for print strings in the console.
Line 5: default the `main` function. Main function will be the entry point of this app.  
Line 6: Use the `fmt` package to print "Hello world!".  
Line 7: closing main function.  

Now we can run the app. Switch to your terminal window (shortcut: control+`) and run the app using:  

```bash
go run .
# or build the app first and then run the binary
go build .
./helloworld
```

In the console, you should see `Hello world!` printed out after running the application.

Congratulations, every thing works perfectly. You have set up Go development environment so far and you are ready to start the journey to Pro.  

In the part II, you will learn the basic syntax of Go. Stay tuned and I will see you in the next stop.
