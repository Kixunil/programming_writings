# Packaging is not dead

*Note: this is a programming article but non-programmers should find it valuable too.*

## Introduction

Over last several years I've observed some push against distribution packages.
People avoid them and prefer to install libraries, statically-linked binaries or containers directly from upstream.
Some applications even actively advise against distribution packages.
They provide these reasons:

* Some distributions don't offer the dependency/application
* Many things in distribution packages are old
* Dependency hell - maybe the distribution offers the dependency but the application doesn't work with it
* Unreliability because of moving parts - some dependency changes without you knowing in a way that causes problems

I can completely see why someone would be bothered by these and why people choose to bypass distributions.
There's a time and place for doing so.
And I suspect I even forgot some reasons.
But it seems to me people often forget about the other side - the advantages of packaging and disadvantages of not packaging.
I wonder how many people can at least enumerate what those are even if they are not worth it for them.
Can you?
Consider pausing here and trying.

In this article I will try to list all that I can think of with a detailed explanation of each.
I will argue using packages is still useful today and worth considering.
While I do not intend to convince everyone to use packages all the time I hope I can persuade some people to give packages a chance.

## Historical context

As you all probably know a long time ago storage capacity of comupters was much lower than today and so was the network speed.
Among other optimizations it made sense to try and reduce consumption of these resources.
One of the most useful optimizations is just avoiding doing the same thing multiple times.
Not storing same file multiple times, not transferring it over the network multiple times.
That's how dynamic linking was invented.

Thanks to dynamic linking two applications that do the same thing, e.g. image editor and a web browser both decoding images,
can share the same piece of code to do so.
Thus this piece of code, called a library and stored on the disk as a file was only present on the system once.
But not just that.
Thanks to a clever trick (memory mapping) it could be in RAM once as well.
This was a big deal.
And in principle it could be downloaded just once.
This is where the DLL hell comes from though.

The DLL hell, named after the library extension, describes the situation often seen on Windows.
A certain application would need a library but it wouldn't carry it itself relying on the user to install it separately.
Not only was it a bad UX but it made uninstallation even harder.
You never knew if a library is still needed by some application or if all applications that needed it previously were uninstalled and the library can be removed as well.
To this day Windows doesn't provide a solution for this but since the capacities of memories and network increased people just started always bundling the libraries with their applications as the original problem didn't exist anymore.

On the other hand, Linux distributions provided a solution:
they made software that keeps track of all libraries or non-library dependencies and automatically installs them from the Internet when needed or uninstall them when not.
They called the individual components that can be installed "packages" and were the first "App stores" before the first widely-known App Store existed.
This sounds great though the above mentioned problems cropped up eventually and thus some people gave up on them.
However coincidentally these packaging systems increased in complexity over time and provided more advantages than just computer resource savings.

## The advantages of packaging

### Security updates

The first and IMO very important advantage of dynamic linking is making security updates and some bugfixes easier.
If you have a web browser and an image editor both using the same image decoding library and a bug is found that makes it possible to take over your system by sending you a corrupt image, you can just update the dynamically linked library rather than updating both the browser and the image editor.

And yes, part of this is resource saving again but this time it's actually higher.
With just installation you only save the cost of the library which is generally small
but with upgrades you avoid whole cost of re-downloading the whole image editor and the browser which are typically large applications.
Given that these are often offered for free, saving the server costs can be interesting.

But it's not only that.
In case of such update, all *statically* linked applications must be rebuilt - a lengthy process usually tiggerred by a human.
And suddenly every single application is responsible for doing this and shipping their application,
including regularly monitoring all of their dependencies checking if they published anything important.
A small software company or a hobbyist doesn't have the resources to make the update quickly?
Bad luck, their application will stay vulnerable for longer.
Or maybe they will prioritize the security update but it means you'll get a feature you've been waiting for later.

Further, there is not easy way to figure out which dependencies a statically linked application uses.
If a new vulnerability arises how do you know if you've updated all the software you use?
Was your favorite game not update because they don't use the library or because they forgot/missed it/didn't have the time?
While it is theoretically *possible* there's no standardized way of solving this and unless you're a programmer, you're technically incapable of doing so.
Even programmers don't bother doing this.

### Integration of applications

With the rise of statically linked binaries, containers and so on, there's also a push towards isolating applications from the operating system or other applications.
Surely, other components bring sources of instability.
This other application changed a file in an unexpected way causing our application to crash?
That's bad, let's prevent applications from interacting with each-other.
And while this approach works well on servers it's not really great in user-facing apps.

The user facing applications often *need* interaction.
You made a nice drawing in your image editor and want to e-mail it to your friend?
That's an interaction.
You want to copy-paste?
That's an interaction.
You want to actually display anything?
That's an interaction - with the window manager&co.
Try making a user-facing application that truly doesn't interact with anything and tell me how many users you got (and kept).
I myself have encountered a few applications that had broken or unimplemented copy-paste and stopped using them.
How many people would do the same?
It's also note worthy that Apple gets praise for their device working together reliably.
Do you think their users don't care about it?

But indeed, willy-nilly interacting can be a problem.
Can my application put a file here?
How do I register file extension?
How does an application gain rights to some external devices?
Does the other, optional application even exist?
How do I connect to it?

Many of these things are, at least to a degree, solved by packaging.
If you target a Linux distribution you can read their documentation on ho to do things and if any of those need additional dependencies.
Some of the interactions happen during installation - such as registering the file extensions.
And while it could be correctly pointed out that packaging itself doesn't solve this, some distributions just picked an API that relies on it and needs to be respected.
Also it makes some things much easier.
(For instance in Debian a tool called debhelper can help you setup and manage a service pretty easily.)

### Actually, resources are still a problem

Yeah, not the libraries.
But there are other kinds of dependencies that are still pretty big or their duplication has other annoying costs.
As a Bitcoiner I can easily give you some examples: The timechain, various wallets.
As a maintainer of [Debian cryptoanarchy packages](https://github.com/debian-cryptoanarchy/cryptoanarchy-deb-repo-builder) I still get questions like "Is it possible to use pruning? I don't have enough space/enough money to buy more space."
And that one includes the chain *just once*.
Now imagine if every single Bitcoin application packaged its own Bitcoin Core.
Even with pruning, Bitcoin Core still takes a lot of CPU and IO resources to sync.
It'd be a nightmare.
(BTW this is also part of why Blockstream satellite is unsupported in CADR - it is a fork of Core.)

But it's not just that single special application.
There's also electrs and NBxplorer.
Further, imagine having multiple instances of a Lightnin Network node.
Having to manage channels for each of them and move money over public network from one of your own wallets to the other.
You'd burn a lot of money on fees not to mention angering toximalists about the chain use. ;)
And once you have more than one-two applications that have this property, why not just reuse the existing dependency-solving system?

### Reinventing the wheel

This is another trend I see: the authors of different applications see the issues I write about and rather than using an existing solution they start
inventing their own app stores or hacked-together shell scripts (Nextcloud, BTCPayServer, RaspiBlitz, ...).
They have to re-write the algorithms and deal with things someone else already solved.
People developing `apt`, `yum`, etc already did a ton of work figuring out various technical problems around it including software signing, upgrades, uninstallation...
And even if reinventing them is doable,
they will never achieve the level of system integration a distribution packaging is capable of since they operate on a totally different level under an unprivileged user.
(And no, I don't want to give publicly-facing web apps full root access, I'm not crazy.)

As a simple example: there's a video convertor Nextcloud app that requires system dependency (ffmpeg) to work.
I guess their author didn't want to rewrite all the video codes and formats in PHP. :)
And I guess shipping a platform-dependent binary in Nextcloud also isn't possible.
So the app requires the administrator to install a system dependency manually.
That's not a nice UX!
If the system packaging was used instead one would just install it in one go in one place and not have to deal with these things.

## We're not actually getting rid of dependency problems

Here's a funny idea: try writing a high-quality useful application with literally zero dependencies.
No libraries, no operating system, no services.
You can't do it.
Nobody can.

Even with all static linking, containers, etc there will still be dependencies - on the kernel, container solution, etc.
And if we want our applications to be useful very likely on other things like public APIs and other applications as discussed above.
So running from them is not a solution.
Limiting them can be helpful in the short term but ultimatly brings lower value to the user.

In my mind, the common "Just use Docker" can be changed to "Just use Debian" - it suggests that you can deal with dependencies by adding another dependency.
And yes, if everyone just chose one true packaging system and one true operating system many of compatibility problems would go away.
I don't think it'll happen anytime soon.

## Conclusion

I hope you can now see at least some value in distribution packaging.
Even if you ultimately decide to not use it in some cases, hopefully you will in others.
Yes, the new software needs to be developed, the new APIs of libraries need to be tested and yes, those need to bypass the packaging.
And yes, the modern development practices allow releasing things much faster than before making many things feel obsolete.
But when there's time and place for reliable, polished software, packaging can help a lot.

So dear users, consider installing "older" software from a package manager next time.
Maybe you'll appreciate greater reliability, security and UX that way.
Do you really need that feature right now?
Even if you have to become technical to install the app?
Or maybe you need to risk installing a malware?
Sure, sometimes the answer is "yes".
Just know what the alternative is.

And programmers, try to work together with packagers to make your software better.
It's not a lot, the changes are usually pretty small and contained.
Maybe add systemd notify feature or improve file handling.
And if nobody wants to package your software you can still do it on your own.
Such work is indeed pretty complex, I know it first-hand.
That's why I'm developing [a tool](https://github.com/Kixunil/debcrafter) to improve the situation.
Feel free to ask for help or help me with development!
