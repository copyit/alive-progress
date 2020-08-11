[<img align="right" src="https://cdn.buymeacoffee.com/buttons/default-orange.png" width="217px" height="51x">](https://www.buymeacoffee.com/rsalmei)

![alive-progress logo](img/alive-logo.gif)

# alive-progress :)
### A new kind of Progress Bar, with real-time throughput, eta and very cool animations!

[![Maintenance](https://img.shields.io/badge/Maintained%3F-yes-green.svg)](https://gitHub.com/rsalmei/alive-progress/graphs/commit-activity)
[![PyPI version](https://img.shields.io/pypi/v/alive-progress.svg)](https://pypi.python.org/pypi/alive-progress/)
[![PyPI pyversions](https://img.shields.io/pypi/pyversions/alive-progress.svg)](https://pypi.python.org/pypi/alive-progress/)
[![PyPI status](https://img.shields.io/pypi/status/alive-progress.svg)](https://pypi.python.org/pypi/alive-progress/)
[![Downloads](https://pepy.tech/badge/alive-progress)](https://pepy.tech/project/alive-progress)

Ever found yourself in a remote ssh session, doing some lengthy operations, and every now and then you felt the need to hit [RETURN] just to ensure you didn't lose the connection? Ever wondered where your processing was in, and when would it finish? Ever needed to *pause* the progress bar for a while, return to the python REPL for a manual inspection or fixing an item, and then *resume* the process like it never happened? I did...

I've started this cool progress bar thinking about all that, the **Alive-Progress**! :)

![alive-progress demo](img/alive-demo.gif)


I like to think of it as a new kind of progress bar for python, as it has among other things:

- a **cool live spinner**, which clearly shows your lengthy process did not hang or your ssh connection did not drop;
- a **visual feedback** of your processing _current speed/throughput_, as the spinner runs faster or slower with it;
- an **efficient** multi-threaded bar, which updates itself at a fraction of the actual speed (1,000,000 iterations per second equates to only 60 updates per second) to keep **CPU usage low** and avoid terminal spamming (you can even calibrate this);
- an **ETA** (expected time of arrival), with a smart _exponential smoothing algorithm_ that shows the remaining processing time in the most friendly way;
- a **print() hook** and (📌 new) **logging support**, which allows print statements and logging messages to work _effortlessly_ in the midst of an animated bar, automatically cleaning the screen and even enriching them with the current bar position when they occurred;
- a **nice receipt** is printed after your processing has finished, with the statistics of that run including the elapsed time and observed throughput;
- it detects and looks different in **under and overflows**, enabling you to track your desired count, not necessarily the actual iterations;
- it automatically detects if there's an **allocated tty**, and if there isn't (like in a shell pipeline), only the final receipt is printed, so you can safely include it in any code and rest assure your log file won't get thousands of progress lines;
- you can **pause** it! I think that's an unprecedented feature for ANY progress bars, no one has ever done that! It's incredible to be able to get the prompt back and manually fix some items while **running inside a progress bar** context, and get it back like it had never stopped whenever you want;
- it is **customizable**, with a growing smorgasbord of different bar and spinner styles, as well as several factories to easily generate yours!


> ### 📌 New in 2.0 series!
> - bars engine revamp, with invisible fills and advanced support for multi-char tips, which gradually enter and leave the bar
> - spinners engine revamp, with standardized factory signatures, improved performance and new features: smoother bouncing spinners with an additional frame at the edges and optimized scrolling of text messages (they go slower and pause for a moment at the edges), new animation mode in compound spinners which can now run alongside or in sequence, nicer effect in alongside compound spinners using weighted spreading over the available space, smoother animation in scrolling spinners when the input is longer than available space
> - new builtin spinners, bars and themes, using the new animation features
> - showtime now has themes! and is more concise, does not wrap screen and can filter patterns
> - `bar()` handle now supports absolute and relative positioning in all modes
> - improved logging for files, enriched as print
> - new advanced configuration for `unknown` bar, supporting spinner names, spinner factories and unknown bar factories
> - uses `time.perf_counter()` high resolution clock
> - improved `print_chars()` utility
> - requires python 3.6+ (and officially supports python 3.9 and 3.10)

---

## Get it

Just install with pip:

```bash
$ pip install alive-progress
```


## Awake it

Open a context manager like this:

```python
from alive_progress import alive_bar

with alive_bar(total) as bar:  # declare your expected total
    for item in items:         # iterate as usual over your items
        ...                    # process each item
        bar()                  # call after consuming one item
```

And it's alive! 👏

In general lines, just retrieve the items, enter the `alive_bar` context manager with their total, and just iterate/process normally, calling `bar()` once per item! It's that simple! :)


## Grasp it

- the `items` can be any iterable, like for example a queryset;
- the first argument of the `alive_bar` is the expected total, so it can be anything that returns an integer, like `qs.count()` for querysets, `len(items)` for iterables that support it, or even a static integer;
- the `bar()` call is what makes the bar go forward -- you usually call it in every iteration after consuming an item;
- the `bar()` call returns the current count or percentage.

> You can get creative! Since the bar go forward only when you call `bar()`, it is _independent of the loop_! So you could use it to monitor anything you want, like only when you find something interesting or even more than once in the same iteration, and at the end you'll get to know how many of those events there were!

So, you could even use it without any loop, like for example:

```python
with alive_bar(3) as bar:
    corpus = read_big_file(file)
    bar()  # file read, tokenizing
    tokens = tokenize(corpus)
    bar()  # tokens ok, processing
    process(tokens)
    bar()  # we're done! 3 calls with total=3
```

<details>
<summary>Oops, there's a caveat using without a loop...</summary>

> Note that if you use `alive-progress` without a loop it is your responsibility to equalize the steps! They certainly do not have the same durations, so the ETA can be very misleading. Since you are telling the `alive-progress` there's three steps, when the first one gets completed it will understand 1/3 or 33% of the whole processing is complete, but reading that big file can actually be much faster than tokenizing or processing it.
> <br>To improve on that, use the **manual mode** and increase the bar by different amounts at each step!
>
> You could use my other open source project [about-time](https://github.com/rsalmei/about-time) to easily measure the duration of each step, then dividing them by the total time and obtaining their percentages. Accumulate those to get the aggregate percentages and that's it! Just try to use some representative inputs, like ordinary big files someone could actually use. Something like:
>
> ```python
> from about_time import about_time
>
> with about_time() as t_total:             # this about_time will measure the whole time of the block.
>     t1 = about_time(read_big_file, file)  # the three original steps are wrapped in about_time calls.
>     t2 = about_time(tokenize, t1.result)
>     t3 = about_time(process, t2.result)
>
> print(f'percentage1 = {t1.duration / t_total.duration}')
> print(f'percentage2 = {t2.duration / t_total.duration + percentage1}')
> print(f'percentage3 = {t3.duration / t_total.duration + percentage2}')  # the last should always be 1.
> ```
>
> Then you can use these percentages to improve the original code:
>
> ```python
> with alive_bar(3, manual=True) as bar:
>     corpus = read_big_file()
>     bar(0.01)  # bring the percentage of the first step
>     tokens = tokenize(corpus)
>     bar(0.3)  # bring the percentage of the first + second steps
>     process(tokens)
>     bar(1.0)  # the last is always 100%, we're done!
> ```
> ---
</details>


## Modes of operation

Actually, the `total` argument is optional. Providing it makes the bar enter the **definite mode**, the one used for well-bounded tasks. This mode has all statistics widgets `alive-progress` has to offer: count, throughput and eta.

If you do not provide a `total`, the bar enters the **unknown mode**. In this mode, the whole progress bar is animated like the cool spinners, as it's not possible to determine the percentage of completion. Therefore, it's also not possible to compute an eta, but you still get the count and throughput widgets.

> The cool spinner are still present in this mode, and they're both running their own animations, concurrently and independently of each other, rendering a unique show in your terminal! 😜

Then you have the **manual modes**, where you get to actually control the bar position. It's used for processes that only feed you back the percentage of completion, so you can inform it directly to the bar.

Just pass a `manual=True` argument to `alive_bar` (or `config_handler`), and you get to send your own percentage to the very same `bar()` handler! For example to set it to 15%, you would call `bar(0.15)`, which is 15 / 100, as simple as that.
<br>Call it as frequently as you need, the refresh rate will be asynchronously computed as usual, according to current progress and elapsed time.

And do provide the `total` if you have it, to get all the same count, throughput and eta widgets as the _definite mode_! If you don't, it's not possible to infer the count widget, and you'll only get simpler versions of the throughput and eta widgets: throughput is only "%/s" (percent per second) and ETA is only to get to 100%, which are very inaccurate, but better than nothing.

> But it's quite simple: Do not think about which mode you should use, just always pass the expected `total` if you know it, and use `manual` if you need it! It will just work the best it can! 👏\o/

To summarize it all:

| mode | completion | count | throughput | eta | overflow and underflow |
|:---:|:---:|:---:|:---:|:---:|:---:|
|     definite        | ✅<br>automatic  | ✅             | ✅            | ✅          | ✅ |
|     unknown         | ❌               | ✅             | ✅            | ❌          | ❌ |
| manual<br>bounded   | ✅<br>you choose | ✅<br>inferred | ✅            | ✅          | ✅ |
| manual<br>unbounded | ✅<br>you choose | ❌             | ⚠️<br>simpler | ⚠️<br>rough | ✅ |


### The `bar()` handlers

The `bar()` handlers support both relative and absolute semantics, but the default one is dependent on the mode, to make them easier to use:
- in **definite and unknown** modes, the default is **relative positioning**, so you can just call `bar()` to increment count by one — optionally send another increment like `bar(5)` to skip the count by 5 in one step;
- in **manual** modes, the default is **absolute positioning**, so you can just call `bar(0.35)` to set the bar directly to 35% — that means the argument is mandatory here;
- they always return the updated count/percentage value.

(📌 new) Now all modes support relative or absolute semantics! They're not that common, but if you ever need them...
<br>Just include a `relative=True` to use relative, or `relative=False` to use absolute.

So, to directly set the bar to count 48 in **definite and unknown** modes, just call `bar(48, relative=False)`.
<br>Or to increase the percentage by 24% in **manual** modes, just call `bar(0.24, relative=True)`.

> You can get creative here too! Since you can set the bar to whatever position you want, you could make it go backwards or even act like a gauge of some sort!


## Styles

Wondering what styles does it have bundled? It's `showtime`! ;)

![alive-progress spinners](img/showtime-spinners.gif)

Actually I've made these styles just to put to use all combinations of the factories I've created, but I think some of them ended up very very cool! Use them at will, or create your own!

The `showtime` exhibit have an optional parameter to choose the show, take a look at the bars one! ;)

![alive-progress bars](img/showtime-bars.gif)

(📌 new) And there's even a themes `showtime`, how cool is that?

![alive-progress themes](img/showtime-themes.gif)

The `showtime` exhibit accepts that optional parameter to choose the show — `Show.SPINNERS` (default), `Show.BARS` or `Show.THEMES` — and the following keyword parameters to modify them:
- **fps**: the refresh rate per second to update the screen, default is 15;
- **length**: the length of the bars, default is 40;
- **pattern**: a filter to choose what to display.

For example to get a marine show, you can `showtime(pattern='boat|fish|crab')`:

![alive-progress filtered spinners](img/showtime-marine-spinners.gif)

> You can also access these shows with the commands `show_bars()`, `show_spinners()`, and `show_themes()`, with the same keyword parameters.

And finally there's also a utility called `print_chars()`, to help finding that cool character to put in your customized spinners or bars, or to determine if your terminal do support unicode characters.


## Displaying messages

While inside an `alive_bar` context, you can effortlessly display messages with:
- the usual `print()` statement and even `logging` framework, where `alive_bar` properly cleans the line, prints or logs an enriched message (including the current bar position) and continues the bar right below it;
- the (📌 new) `bar.text('message')`, which sets a situational message right within the bar, usually to display something about the items being processed or the phase the processing is in. Use at will.

![alive-progress printing messages](img/print-hook.gif)


## Appearance and behavior

There are several options to customize appearance and behavior, most of them usable both locally (inside an `alive_bar` context) or globally (which auto-activates in all `alive_bar` contexts).

These are the few ones that only make sense locally:
- `title`: an optional bar title, always visible, that represents what is being processed;
- `calibrate`: tweaks the fps calibration engine (more details [here](#advanced))

And these can be used anywhere [default values in brackets]:
- `length`: [`40`] the number of characters to render the actual bar widget;
- `spinner`: the spinner style to be rendered next to the bar
<br>   ↳ accepts the name of a predefined spinner, or a custom spinner factory;
- `bar`: the bar style to be used in all bounded modes
<br>   ↳ accepts the name of a predefined bar, or a custom bar factory;
- `unknown`: the bar style to be used in the unknown mode (where the whole bar is a big spinner)
<br>   ↳ accepts the name of a predefined spinner, a custom spinner factory or a custom unknown factory;
- `theme`: [`'smooth'`] the set of spinner, bar and unknown to be used
<br>   ↳ accepts the name of a predefined theme;
- `force_tty`: [`False`] runs animations even without a connected tty (more details [here](#advanced));
- `manual`: [`False`] set to enter a manual mode, and manually control the bar percentage;
- `enrich_print`: [`True`] enriches with the bar position all print() and logging messages;
- `title_length`: [`0`] aligns title to this length, or 0 for not align
<br>   ↳ if title is longer than this number, it will be truncated, and an ellipsis will appear at the end.

To use them locally just send as keyword arguments to `alive_bar`:

```python
from alive_progress import alive_bar

with alive_bar(total, title='Processing', length=20):
    ...
```

To use them globally, send them to `config_handler`, and any `alive_bar` created after that will include those options! And you can mix and match them, local options always have precedence over global ones:

```python
from alive_progress import alive_bar, config_handler

config_handler.set_global(length=20, spinner='wait')

with alive_bar(total, bar='blocks', spinner='twirl'):
    # the length is 20, the bar is 'blocks' and the spinner is 'twirls' ;)
    ...
```


## Create your own animations

You can assemble your own spinners and bars! All of the major components are individually customizable, with builtin support for a plethora of special effects like frames, scrolling, bouncing, delayed and compound spinners! Get creative!

The animations are engineered by very advanced generators, defined by factories of factories of generators within closures! The first level receives and processes the styling parameters to create the actual factories, encapsulating them in a closure. These are the actual objects you'll send to customize the bar styles in both `alive_bar` and `config_handler`. Internally they still receive other operating parameters (like their desired rendition lengths), and are encapsulated in yet another closure, to finally engineer the infinite generators used by the animations you see on the screen.

These generators are capable of several different animation cycles, for example a bouncing one has a cycle to the right and another to the left. They continually yield the next animation frame in a cycle until they're exhausted, which enables the next one, but does not start it!
<br>That has all kinds of cool implications: the cycles can have different frame counts, different screen lengths, they do not need to be synchronized, they can create long different sequences by themselves, they can cooperate with each other to play cycles in sequence or simultaneously, and I can amaze you displaying several at once on the screen without any interferences! It's almost like they are _alive_! 😉 ===> Yes, that's where this project's name came from.
<br>Seriously, I think they're the greatest breakthrough I've achieved in this project, if I say so myself. Take a look at them in the [spinners module](https://github.com/rsalmei/alive-progress/blob/master/alive_progress/animations/spinners.py).

The types of factories I've created are:
- `frames`: draw any sequence of characters, that will be played frame by frame in sequence;
- `scrolling`: pick a frame or a sequence of characters and make them flow smoothly from one side to the other, hiding behind or wrapping upon invisible borders — if using a sequence, generates several cycles of distinct characters;
- `bouncing`: similar to `scrolling`, but make the animations bounce back and return, hiding behind or immediately bouncing upon invisible borders — supports several interleaved cycles too;
- `compound` get a handful of factories and play them alongside simultaneously or one after the other sequentially! why choose if you can have them all?
- `delayed`: get any other factory, and copy it multiple times, increasingly skipping some frames on each one! very cool effects are made here;

Here's an example on how to use them:

![alive-progress custom animations](img/alive-custom-animations.gif)


## Advanced

You should now be completely able to use `alive-progress`, have fun!
<br>If you've appreciated my work and would like me to continue improving it, you can donate me a beer or a coffee! I would really appreciate that 😊! Thank you!

And if you want to do even more, exciting stuff lies ahead!

<details>
<summary><strong><em>You want to calibrate the engine?</em></strong></summary>

> ### FPS Calibration (📌 new)
>
> The `alive-progress` bars have a cool visual feedback of the current throughput, so you can instantly **see** how fast your processing is, as the spinner runs faster or slower with it.
> For this to happen, I've put together and implemented a few fps curves to empirically find which one gave the best feel of speed:
>
> ![alive-progress fps curves](img/alive-bar_fps.png)
> (interactive version [here](https://www.desmos.com/calculator/ema05elsux))
>
> The graph shows the logarithmic (red), parabolic (blue) and linear (green) curves, these are the ones I started with. It was not an easy task, I've made hundreds of tests, and never found one that really inspired that feel of speed I was looking for. The best one was the logarithmic one, but it reacted poorly with small numbers.
> I know I could make it work with a few twists for those small numbers, so I experimented a lot and adjusted the logarithmic curve (dotted orange) until I finally found the behavior I expected. It is the one that seemed to provide the best all around perceived speed changes throughout the whole spectrum from units to billions.
> That is the curve I've settled with, and it's the one used in all modes and conditions. In the future and if someone would find it useful, that curve could be configurable.
>
> Well, the default `alive-progress` calibration is **1,000,000** in bounded modes, i.e., it takes 1 million iterations per second for the bar to refresh itself at 60 frames per second. In the manual unbounded mode it is **1.0** (100%). Both enable a vast operating range and generally work really well.
>
> For example, take a look at these very different calibrations running the very same code, at the very same speed! Notice the impact this parameter actually have, it isn't only the spinner but the whole line, complete with the bar rendition and all widgets:
>
> ![alive-progress calibration](img/alive-calibration.gif)
>
> > So, if your processing hardly gets to 20 items per second and you think `alive-progress` is rendering sluggish, you could increase that sense of speed by calibrating it to let's say `40`, and it will be running waaaay faster... Actually it is better to leave some headroom and calibrate it to something between 50% and 100% more, and then tweak it from there to find the one you like the most! :)
>
> ---
</details>

<details>
<summary><strong><em>Oh you want to stop it altogether!</em></strong></summary>

> ### The Pause Mechanism
>
> Why would you want to pause it, I hear? To get to manually act on some items at will, I say!
> <br>Suppose you need to reconcile payment transactions (been there, done that). You need to iterate over thousands of them, detect somehow the faulty ones, and fix them. This fix is not simple nor deterministic, you need to study each one to understand what to do. They could be missing a recipient, or have the wrong amount, or not be synced with the server, etc, it's hard to even imagine all possibilities. Typically you would have to let the detection process run until completion, appending to a list each inconsistency found, and waiting potentially a long time until you can actually start fixing them. You could of course mitigate that by processing in chunks or printing them and acting in another shell, but those have their own shortcomings.
> <br>Now there's a better way, pause the actual detection for a moment! Then you have to wait only until the next fault is found, and act in near real time!
>
> To use the pause mechanism you must be inside a function, to enable the code to `yield` the items you want to interact with. You should already be using one in your code, but in the ipython shell for example, just wrap the `alive_bar` context inside one. Then you just need to enter the `bar.pause()` context!! Something like `with bar.pause(): yield transaction`.
>
> ```python
> def reconcile_transactions():
>     qs = Transaction.objects.filter()  # django example, or in sqlalchemy: session.query(Transaction).filter()
>     with alive_bar(qs.count()) as bar:
>         for transaction in qs:
>             if not validate(transaction):
>                 with bar.pause(): yield transaction
>             bar()
> ```
>
> That's it! Then you can use it in any code or even ipython! Just call the reconcile function to instantiate the generator and assign it to `gen` for example, and whenever you want another transaction to fix, call `next(gen, None)`! The progress bar will pop in as usual, but as soon as an inconsistency is found, the bar pauses itself and you get the prompt back with a transaction! It's almost magic! 😃
>
> ```text
> In [11]: gen = reconcile_transactions()
>
> In [12]: next(gen, None)
> |█████████████████████                   | 105/200 [52%] in 5s (18.8/s, eta: 4s)
> Out[12]: Transaction<#123>
> ```
>
> You can then use `_12` ipython's shortcut to get the transaction, if you don't like that just assign it with `trn = next(gen, None)`, and you're set up as well to fix that `trn` at once!
> <br>When you're done, revive the detection process with the same `next` as before... The bar reappears **exactly like it had stopped** and continues on the next item like nothing happened!! Nice huh :)
>
> ```text
> In [21]: next(gen, None)
> |█████████████████████                   | ▁▃▅ 106/200 [52%] in 5s (18.8/s, eta: 4s)
> ```
>
> ---
</details>

<details>
<summary><strong><em>Those astonishing animations refuse to display?</em></strong></summary>

> ### Forcing animations on non-interactive consoles
>
> There are ttys that do not report themselves as "interactive", which are valid for example in shell pipelines "|" or headless consoles. But there are some that do that for no good reason, like Pycharm's python console for instance.
> The important thing is, if a console is not interactive, `alive_bar` disables all the animations and refreshes, as that could break some output, and prints only the final receipt.
> So if you are in a safe environment like Pycharm's and would like to see `alive_bar` in all its glory, I've included a `force_tty` argument!
>
> ```python
> with alive_bar(1000, force_tty=True) as bar:
>     for i in range(1000):
>         time.sleep(.01)
>         bar()
> ```
>
> You can also set it system-wide in `config_handler`, then you won't need to pass it anymore.
>
> Do note that Pycharm's console is heavily instrumented and thus has more overhead, so the outcome may not be as fluid as you would expect. To see `alive_bar` animations perfectly, always prefer a full-fledged terminal.
>
> ---
</details>


## Interesting facts

- This whole project was implemented in functional style;
- It does not declare even a single class;
- It does not have any dependency;
- It uses extensively (and very creatively) python _closures_ and _generators_, with _infinite loops_ inside, they're in several modules — look for instance the [spinners module](https://github.com/rsalmei/alive-progress/blob/master/alive_progress/animations/spinners.py), the [exhibit module](https://github.com/rsalmei/alive-progress/blob/master/alive_progress/styles/exhibit.py) and the [spinner_player function](https://github.com/rsalmei/alive-progress/blob/master/alive_progress/animations/utils.py) 😜.


## To do

- reset a running bar context
- dynamic bar width rendition, listening to changes in terminal size
- improve test coverage, currently at 77% branch coverage, working to achieve 100%
- enable multiple simultaneous bars, for nested or multiple activities
- create a contrib system, to allow a simple way to share users' spinners and bars styles
- jupyter notebook support
- support colors in spinners and bars
- any other ideas welcome!

<details>
<summary>Already done 👍</summary>

> - create an unknown mode for bars (without a known total and eta)
> - implement a pausing mechanism
> - change spinner styles
> - change bar styles
> - include a global configuration system
> - create customizable generators for scrolling, bouncing, delayed and compound spinners
> - create an exhibition for spinners and bars, to see them all in motion
> - include theme support in configuration
> - soft wrapping support
> - hiding cursor support
> - python logging support
> - exponential smoothing of ETA time series
> - create an exhibition for themes
>
> ---
</details>


## Python versions End of Life notice

The `alive_progress` framework starting from version 2.0 does not support Python 2.7 and 3.5 anymore.
<br>If you still need support for them, you can always use the versions 1.x, which are also full-featured and do work very well, just:

```bash
$ pip install -U "alive_progress<2"
```

> If you put this version as a dependency in a requirements.txt file, I strongly recommend to put `alive_progress<2`, as this will always fetch the latest release of the v1.x series. That way, if I ever release a bugfix for it, you will get it the next time you install it.


## Changelog highlights:
- 2.0.0: bars engine revamp, with invisible fills and advanced support for multi-char tips, which gradually enter and leave the bar; spinners engine revamp, with standardized factory signatures, improved performance and new features: smoother bouncing spinners with an additional frame at the edges and optimized scrolling of text messages (they go slower and pause for a moment at the edges), new animation mode in compound spinners which can now run alongside or in sequence, nicer effect in alongside compound spinners using weighted spreading over the available space, smoother animation in scrolling spinners when the input is longer than available space; new builtin spinners, bars and themes, using the new animation features; showtime now has themes! and is more concise, does not wrap screen and can filter patterns; `bar()` handle now supports absolute and relative positioning in all modes; improved logging for files, enriched as print; new advanced configuration for `unknown` bar, supporting spinner names, spinner factories and unknown bar factories; uses `time.perf_counter()` high resolution clock; improved `print_chars()` utility; requires python 3.6+ (and officially supports python 3.9 and 3.10)
- 1.6.1: fix logging support for python 3.6 and lower; support logging for file; support for wide unicode chars, which use 2 columns but have length 1
- 1.6.0: soft wrapping support; hiding cursor support; python logging support; exponential smoothing of ETA time series; proper bar title, always visible; enhanced times representation; new `bar.text()` method, to set situational messages at any time, without incrementing position (deprecates 'text' parameter in `bar()`); performance optimizations
- 1.5.1: fix compatibility with python 2.7 (should be the last one, version 2 is in the works, with python 3 support only)
- 1.5.0: standard_bar accepts a `background` parameter instead of `blank`, which accepts arbitrarily sized strings and remains fixed in the background, simulating a bar going "over it"
- 1.4.4: restructure internal packages; 100% branch coverage of all animations systems, i.e., bars and spinners
- 1.4.3: protect configuration system against other errors (length='a' for example); first automated tests, 100% branch coverage of configuration system
- 1.4.2: sanitize text input, keeping \n from entering and replicating bar on screen
- 1.4.1: include license file in source distribution
- 1.4.0: print() enrichment can now be disabled (locally and globally), exhibits now have a real time fps indicator, new exhibit functions `show_spinners` and `show_bars`, new utility `print_chars`, `show_bars` gains some advanced demonstrations (try it again!)
- 1.3.3: further improve stream compatibility with isatty
- 1.3.2: beautifully finalize bar in case of unexpected errors
- 1.3.1: fix a subtle race condition that could leave artifacts if ended very fast, flush print buffer when position changes or bar terminates, keep total argument from unexpected types
- 1.3.0: new fps calibration system, support force_tty and manual options in global configuration, multiple increment support in bar handler
- 1.2.0: filled blanks bar styles, clean underflow representation of filled blanks
- 1.1.1: optional percentage in manual mode
- 1.1.0: new manual mode
- 1.0.1: pycharm console support with force_tty, improve compatibility with python stdio streams
- 1.0.0: first public release, already very complete and mature


## License
This software is licensed under the MIT License. See the LICENSE file in the top distribution directory for the full license text.


## Did you like it?

Thank you for your interest!

I've put much ❤️ and effort into this.
<br>If you've appreciated my work and would like me to continue improving it, you can donate me a beer or a coffee! I would really appreciate that 😊! (the button is on the top-right corner) Thank you!
