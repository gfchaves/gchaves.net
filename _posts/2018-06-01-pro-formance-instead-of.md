---
title: Pro-formance instead of
author: Chaves
date: 2018-04-28 19:00:00 +0000
categories: [Blogging, Tools, Productivity]
tags: [.net, performance, book, .net rocks, review, better code]
---
## Intro
Has [Peter Duncanson](https://twitter.com/peteduncanson) said as a final remark at his talk about ["Back office speed hacks"](https://codegarden18.com/sessions/back-office-speed-hacks/), by the way, a great talk this year at CodeGarden 18 (#cg18), we as developers and in our life, we should think about "pro- " instead of performance... strange right? Not at all. The concept is very simple, is the conjunction about being a professional with performance in our daily operations and tasks. It's all about regarding performance in a professional manner on our hands and with that power carried about pushing everything into an optimal way to have a fast, performant and secure development.

![engine_open](/assets/img/posts/engine_open.jpg)

This past two weeks I read the great book about **"Writing High-Performance .NET Code"** written by [Ben Watson](https://twitter.com/benmwatson). You can find everything related to the book and his code samples here. Let me start with his biggest and important remark about the subject:

>Measure, measure, measure

Sure thing that we can't relate performance without measure data, without having times, behaviors or true knowledge regarding what we have now and how we achieve some performant changes to bring a better solution. Measurement is a hard thing... special in a development process if you don't have the process defined it self. Tools and data are easy to obtain but the relationship between them is more complicated to relate and to understand. Although time is essencial to have the performance in the process, is a way better to have her inside the development already. In my experience, I had noticed that until something or some operation is taking to long or in a worst average time to complete, is the point where we face something regarding a bad performant issue... till then, is a matter that is in somewhat offuscated by the tools, frameworks or the process adopted on the task in our hands. I also know that performance is a very broad concept and it can depend directly on the critical aspect or has a requirement of the solution. Because the change on a development process of the team is a very very hard and it can have so many implications above the individual paygrade, I personally prefer to understand our the performance concepts can be ahead and not after when a test or production phase comes.

Knowing the tools and understanding the internals of what we use is the best way to be prepared and to have a real under the hood knowledge of what is happening and why!? We drive our cars in an auto-pilot concept mode, because we perform that task for a long time, although if we became aware of how it works and what is happening underneath our fingers and feets I'll bet with you that you'll have a better driving style, and you'll be aware of when some mechanical issue arrives and how to be on top of it. The same concept can and should be applied to every system or process that we use. Have the knowledge to be on the control!

## The book 
Back to the track, I've been working with .NET Framework since the 1.1 version launched in 2003 and before that the Visual Basic language was my primary one. Even with 15 years of relationship with .NET I still don't know her at all, especially with the most recent changes being pushed out with some many new APIs and internal changes... It's hard to keep up to date with all of the .NET world when our work is mainly focus with Web development in which is just a portion of the many [architectural components of the .NET Framework](https://docs.microsoft.com/en-us/dotnet/standard/components) itself.

>“Managed code lets mediocre developers write lots of bad code really fast.” - Ben W.

Yes, Ben, it's a really hard truth, and I believe that myself victim of this, and when the time is a critical factor is even easy to lay down on this fact. Shooting myself and my developments. But enough of excuses, it's always time to be self-aware of how the internals behaviors are and know the most common pitfalls and anti-patterns that exist, where they are and how to avoid them. The Umbraco team, have a great article about the common pitfalls and every tool or framework should have in order to everyone be aware when developing or doing the tailoring of the final solution.

The ["Writing High-Performance .NET Code"](http://www.writinghighperf.net/) book is a very good way to start and have a deep dive in on the internals, common pitfalls and the anti-patterns issues that we might encounter at our development course. In the next paragraphs, you can read about some of my personal highlights of the book and some of my remarks.

>"So let's get started" - [Captain Joe](https://www.youtube.com/watch?v=R9oqi6HteJg)
If you want to skip the tips, please see final thoughts section for some acknowledge and usefull links.

## Usefull Tips

>ASP.NET Core is a significant improvement over ASP.NET using the .NET Framework. If you want high-performance web serving, it is worth it to adopt ASP.NET Core. - Ben W.

I truly believe in this fact, and the data available does show very significant improvements regarding the performance and the simplicity of it. Although it's a "new-young" framework, last week the [version 2.1](https://blogs.msdn.microsoft.com/dotnet/2018/05/30/announcing-net-core-2-1/) was launched with another set of features and improvements. The years of the very experience .NET team at Microsoft had paid off the data measured and how improvements and patches are being applied, even with all the issues regarding the .NET version between 4.6.1 and 4.7.2. I also know that the hard change to start with .NET Core are the platforms that we as developers depend on. For instance, at our agency, almost work is related with Umbraco CMS platform and we're still stuck with the regular ASP.NET version although the core team is starting moving forward with the core version. But for everything else is a must and [benchmarks](https://blogs.msdn.microsoft.com/dotnet/2018/04/18/performance-improvements-in-net-core-2-1/) of JIT and web servers show the numbers. And about security? Every developer should read this article to be aware of the facts that are known.

>It is very difficult to fix a poorly written application retroactively if it has a fundamentally flawed architecture from an efficiency perspective.

That's a fact that every developer should support their code designs and feature with patterns and validate best and worst case scenarios with higher and experience team members before start code it. After many hours of development, we can perform changes that can bring a way better solution but there are some fundamentals that are very hard to change that don't imply a change of structural pillars of the code that can be very costly. Don't start coding our selling any solution before some architectural validations, and if you still have doubts, there are many online resources that can help you to avoid the famous "common pitfalls".

>You have likely heard the phrase, coined by Donald Knuth, “Premature optimization is the root of all evil.” The context of the quote is in determining which areas of your program are actually important to optimize.

Tooling and measurements are very critical to find bottlenecks indeed. but the most common tools that we use can perform a real help on this. Today almost every framework and powerful IDEs have code analysis, query analysis and you should rely upon first with them. It's very easy to perform LINQ data queries but is also very easy to shoot yourself and have poor performance and large memory or CPU consumption. Know what your code is doing under the hood and try to make a measurement of it. Have you ever tried to use a profiling tool? or even try to load your application under poor connection speed? Or with a non-regular developer workstation? Scaling code with issues is also scaling costs... cloud solutions shouldn't be an excuse.

>Percentiles values are usually far more important for high-availability services. The higher availability you require, the higher percentile you will want to track. Usually, the 99th percentile is as high as you need to care about, but if you deal in a truly enormous volume of requests, 99.99th, 99.999th, or even higher percentiles will be important. Often, the value you need to be concerned about is determined by business needs, not technical reasons.

Know the business that your solution is trying to help, and the critical business factors are very essential for a successful solution. No matter that your application is very fast and not solving the key values that are in the very business interests. It's hard to pass along the developers the business value, although you should care first about them or at least try to know the key factors.

>Context Switch: The process of saving and restoring the state of a thread or process. Because there are usually more running threads than available processors, there are often many context switches per second. These are pure overhead, so fewer is better, but it is difficult to know what an optimal absolute value should be.

If the context switch is hard for our mind when performing multi-task, also is also very hard for our code. We should be aware of the costs of the switch and look up for simple and reliable patterns to help avoid unnecessary code complexity.

>WinDbg is not that interesting for managed code. To work with managed processes effectively, you will need to use .NET’s SOS extensions, which ship with each version of the .NET Framework. A very handy SOS reference cheat sheet is located at [here](https://docs.microsoft.com/dotnet/framework/tools/sos-dll-sos-debugging-extension)
WinDbg is a very useful tool and has been around for so many years. I've known some former colleagues at Microsoft, that they only use this tool on their customer support functions. It's a hard one, but the output information is way valued.

>Performing accurate code timing is also a useful feature at times. Never use DateTime.Now for tracking performance data. It is just too slow for this purpose. Instead, use the System.Diagnostics.Stopwatch class to track the time span of small or large events in your program with extreme accuracy, precision, and low overhead.

Because of the DateTime offsets I always am in favor of the DateTime.UTCNow usage, and believe or not I also wasn't aware of the costs of the DateTime.Now and it's something that can be easily spread out in the code across the entire application.

>No developer, system administrator, or even hobbyist should be without this great set of tools

It's like carrying a Swiss army pocket knife. It's well-known truth and each one of us should have a toolkit set for it. [LinqPad](https://www.linqpad.net/), for instance, is a very useful tool to use alongside your IDE, to help through and doge the pitfalls of your queries. Choose your own tools and be aware to extract the most of them. Like Ben's book **is a must to be at your hand**, to help out in the edged cases described and to use a proof of concept every __time you need to show to someone why you are spending time regarding your application performance issues.__

>Should you use workstation or server GC? If your app is running on a multi-processor machine dedicated to just your application, then the choice is clear: server GC. It will provide the lowest latency collection in most situations.

Very handful tip, if it's your application scenario, be aware that you can use the server GC to get the most of the resources that are available. Otherwise, a wrong choice could be very painful and low "pro- " one.

>Can I convert some classes to structs so they live on the stack, or as part of another object, and have no per-instance overhead?

Be aware that small classes should be structs, and if you benchmark it there are impactful changes. [Benchmark.net](https://github.com/dotnet/BenchmarkDotNet) can help you out on this. And the samples provided by Ben are very factual on this. It's a very simple refactoring that we can do right on our biggest applications that can bring real benefits right away! 😊

>your code spreads out operations on an object, try to reduce the time between the first and last uses so that the GC can collect the object as early as possible.

Again, [SOLID principles](https://hackernoon.com/solid-principles-made-easy-67b1246bcdf) should be applied to avoid this one. Keep simplicity and separation of concepts. It's easier to read throughout the application code and of course during the maintenance operations. It's very common to see classes that are trying to solve all of the "worlds problems". Once I found out a class with 2k lines of code... and with 100 plus lines of operations code methods. Each time I'm coding a new class and when I think that is "finished" I try to read out every method and bring the solid principles into it. Partial classes, services layers, and the most basic object-oriented principles can help us out on this. Of course, the [software patterns](https://www.amazon.com/Head-First-Design-Patterns-Brain-Friendly/dp/0596007124) (link to another great book) are also very useful to solve the complexity and keep well-structured code.

>Dispose methods and finalizers should never throw exceptions. Should an exception occur during a finalizer’s execution, then the process will terminate. Finalizers should also be careful doing any kind of I/O, even as simple as logging.

Also to keep in mind, and it's easy to forget about this tip. Sometimes the usage of exceptions throughout our application code is a practice but be aware of the usage of the disposable and finalizers with exceptions, I also wasn't aware of the costs that this can have. Thanks .

>A newer option for representing pieces of existing buffers is the `Span<T>` struct. `Span<T>` is still in a pre-release phase at the time of this writing, but it will likely become finalized with the release of C# 7.2 or future upgrades to the runtime.

Is out now :) as a part of Visual Studio 2017 15.5 version. I also heard a lot about this new option and the benefits that it has instead of using ArraySegment and in the book, you can find a great sample of usage and the memory benefits that brings along.

>Despite the risks and limitations, stackalloc is a valuable tool when you want small, dynamically sized arrays in your methods without the overhead of a heap allocation.

I never liked to initialize collections or arrays without their limits. Although the risk of the necessity to modify the range size, it's clear the benefits. So the usage of for small and dynamic arrays should be an option to cover these type of implementations. , it's known recommended that you always specify the range allocation and avoid the dynamic ones.

>One of the biggest classes of optimizations is method inlining, which puts the code from the method body into the call site, avoiding a method call in the first place. Inlining is critical for small methods which are called frequently, where the overhead of a function call is larger than the function’s own code.

This one reminds me the power of the language sugar syntax, that sometimes we tend to use a lot of it and it could be also dangerous one.. so again the power of synchronously knowledge" it's very useful.

>With the ubiquity of multicore processors in today’s computers, even on small devices such as cell phones, the ability to program effectively for multiple threads is a critical skill for all programmers.

Yes, it is, and I also admit that is a personal handicap. It's not easy to stop thinking synchronously and be aware of the power that we can have by using libs like Task Parallel and take the right advantage into our applications and their operations that could be performed under multi-thread contexts.

>Finally, do not buy into the notion of total code “ownership.” Everyone should feel ownership for the entire product. There are no independent, competing kingdoms, and no one should be over-protective of “their” code, regardless of original authorship.

A highly competitive work environments or when you're tied up with that application for so long, it may sometimes have this notion of possession. But, in my opinion, this is more like a team attitude that is represented by each one of us.

In our society, I still see some of this and especially regarding with the code review. I've learned a lot by having my code reviewed by more experienced developers and I also believe that is an essential tool for individual grown and for team growth. Together as a team is way better to review and grow together with the same code patterns and exchanging code issues by sharing experiences. It all comes to that.

## Final thoughts
I have to say if I have a team of developers this book would be a requirement of analysis and study. A lot of good knowledge can be extracted but also a lot of how .NET and their APIs work underneath the JIT, CLR and what is reflected in the IL code. Also, it's a book that every .NET developer can read but also for those who want to understand the differences between their programming languages and the .NET Framework one.

So to be "pro-formance" with performance, it's a required reading. And also is clear that the book should be part of the tools to carry around when we're coding. We're getting to close this generational version of applications and preparing a new approach to a high quality applications that users are demanding, high availability and high performance. We want more, fast and reliable solutions into a newer version of applications, for this particular reason and so many others that are written in the book.

The book ["Writting High-Performance .NET Code"](http://www.writinghighperf.net/) has already a second edition, where the author reflected some changes and improvements. And because of the .NET framework long life, I hope to see more editions or a second book that has more focus on the components such as ASP.NET, WPF, WCF and other that are mostly used by the community.

Ben chooses gears to represent the complexity of the system as a comparable well know mechanical gear. I chose a jetliner engine, where I truly know that it has to be a very high-demanded performance system. Capable to operate under many controversial circustances, but also has to be maintainable for at least three decades regarding his complexity... and do we have software with that time-span to maintain? :)

For last I want to congratulate [Ben Watson](https://twitter.com/benmwatson) for his time and dedication on the book and his great [podcast with .NET Rock folks](https://www.dotnetrocks.com/?show=1545).