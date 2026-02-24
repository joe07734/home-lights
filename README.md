# home-lights
Home light automation using Philips Hue / openhue. This isn't written to be plug-and-play if you want to seriously run it in your home. But there are notes below, and if you're a Ruby programmer or want to learn Ruby, I encourage you to give it a try.

I run it on a linux server as two services, one for full automation and one just for the porch light.

It requires Ruby 3.4+, rbenv, openhue, and curl.

## Development
I stop the services and run it from the command line when working on it.

`% home-lights joe`

This runs home lights automation in the foreground with the joe simulation. This is a simulation of me and a typical day, moving around and turning lights on and off. See `class Joe`.

There's quite a bit you'll want to modify if you want to use this for your home. I don't imagine anyone will, but the CS prof in me will explain it anyway. Starting with `class House`, you'll see the array of ROOMS and LIGHTS. The program uses openhue to talk to my existing Hue light setup. In Hue lingo, a room can contain several lights. The only light I refer to directly is the porch light. You can build out your own home by changing this array, then modifying the Joe sim to reference your rooms. The House class contains the low-level methods for talking to the hue bridge in order to turn lights on and off, as well as query the state of lights.

You'll want to modify the sim for your life. Really as a programmer this is where the fun is. Look at the method `one_day` and the methods it calls. I wanted to write the sim cleanly, clearly, stepwise, so the rest of the code in the class is mostly there to support this style. For example `one_day` encapsulates one day, and calls `wake_up`, `breakfast`, etc. and finally `go_to_sleep`. These make use of the routine `start_activity` which starts an activity at a random time, like hanging out by the fireplace, if the time matches an appropriate range. For example in `wake_up` the line `start_activity(oclock(6) .. oclock(10, 30))` means I wake up between 6 and 10:30am, and later in the method `busy_until(System.now + rand_duration(mins(15) .. mins(45)))` means I take between 15 and 45 minutes waking up. `noodle_until` is an important method that's responsible for moving me around the house until one of the main events happens. There are support methods like `move_to("Kitchen")` and `lights_on("Dining Room")`. It's a Joe sim but really just the parts of Joe that affect lights in the house. I'm curious what you think of this coding style.

I include the option `--no_lights` if I want to run it without openhue. This is best when also used with the following options.

When I'm debugging the sim I often use the options `--scaling` and `--scale_time`. Scaling speeds up the sim and is a multiplier, for example `--scaling 2` runs the sim at twice the wall clock time. I usually use it at 1800, or 1 second of wall clock time is 30 minutes of sim. You can also specify the start time when scaling so that you can also start the sim at a repeatable day and time of day. Ruby's `Time.parse()` is used on the argument to `--scale_time` so lots of formats work, for example `--scale_time '2026-03-11 8:00am'`. It's especially useful for testing, since some elements of the sim are random.

But it's also terrific fun during testing to use scaling without the `--no_lights` option. That is, with an entire day simulated with lights throughout the house turning on and off at warp speed. Until my girlfriend says stop it.

In `class System` you'll see the wrappers I wrote for time routines `now` and `sleep` to support scaling. In this way the scaling is hidden, and the sim code can use `now` and `sleep` as usual, so long as it calls them on the System class.

`class Log` is a simple logger to stdout that automatically timestamps lines and understands the verbose flag.

There's also:

`% home-lights porch`

Which runs just the porch light controller. It queries The Internet for the local sunset time and turns the porch on every day, and off at 11:30pm.

This is implemented in `class Porch`. If you're imagining using this yourself, imagine knowing your precise lat/long and then putting those numbers in the line that curls api.sunrise-sunset.org. At the top of the class there's a default sunset time in case the internet doesn't work, and the hardcoded porch light off time. When my girlfriend is out late I adjust the porch off time so it'll be on until she gets home.

Both the Joe sim and the porch light code are written to work correctly no matter what time of day the program is started. For example if it's already after sunset the porch light will turn on immediately. If it's around lunch time, Joe will move to the kitchen.






