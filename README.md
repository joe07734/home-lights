# home-lights
Home light automation using Philips Hue / openhue.

Requires Ruby 3.4+, openhue, curl.

I run it on a linux server as two services, one for full automation and one just for the porch light.

I also run it from the command line when testing.

## Development
`% home-lights joe`
Runs home lights automation in the foreground with the joe simulation. This is a simulation of me and a typical day, moving around and turning lights on and off. See `class Joe`.

I include the option `--no_lights` if I want to run it without openhue.

When I'm debugging the sim I often use the options `--scaling` and `--scale_time`. Scaling speeds up the sim and is a multiplier, for example `--scaling 2` runs the sim at twice the wall clock time. I usually use it at 1800, or 1 second of wall clock time is 30 minutes of sim. You can also specify the start time when scaling so that you can also start the sim at a repeatable day and time of day. Ruby's `Time.parse()` is used on the argument to `--scale_time` so lots of formats work, for example `--scale_time '2026-03-11 8:00am'`. It's especially useful for testing, since some elements of the sim are random.

There's quite a bit you'll want to modify if you want to use this for your home. I don't imagine anyone will, but the CS prof in me will explain it anyway. Starting with `class House`, you'll see the array of ROOMS and LIGHTS. In Hue lingo, a room can contain several lights. The only light I refer to directly is the porch light. You can build out your own home by changing this array, then modifying the Joe sim to reference your rooms. The House class also contains the low-level methods for talking to the hue bridge in order to turn lights on and off, as well as query the state of lights.

You'll want to modify the sim for your lifestyle. Really as a programmer this is where the fun is. Look at the method `one_day` and the methods it calls. I wanted to write the sim cleanly, clearly, stepwise, so the rest of the code in the class is mostly there to support this style. For example `one_day` encapsulates one day, and calls `breakfast`, etc. and finally `go_to_sleep`. These call `start_activity` which can start an activity at a random time, like hanging out by the fireplace, if the time matches an appropriate range. For example in `wake_up` the line `start_activity(oclock(6) .. oclock(10, 30))` means I wake up between 6 and 10:30 and `busy_until(System.now + rand_duration(mins(15) .. mins(45)), "Waking up")` means I take between 15 and 45 minutes waking up. `noodle_until` moves me around the house until one of the main events happens. I'm curious what you think.

In `class System` you'll see the wrappers I wrote for time routines `now` and `sleep` to support scaling. In this way the scaling is hidden, and the sim code can use `now` and `sleep` as usual, so long as it calls them on the System class.

`class Log` is a simple logger to stdout that automatically timestamps lines and understands the verbose flag.

`% home-lights porch`
Runs the porch light controller, which queries The Internet for the local sunset time and turns the porch on every day, and off at 11:30pm.

This is implemented in `class Porch`. If you're imagining using this yourself, imagine knowing your precise lat/long and then putting those numbers in the line that curls api.sunrise-sunset.org. There's a default sunset time and the hardcoded porch light off time at the top of the class. When my girlfriend is out late I adjust the porch off time so it'll be on when she gets home.

Both the Joe sim and the porch light code are written to work correctly no matter what time of day the program is started. For example if it's already after sunset the porch light will turn on immediately. If it's around lunch time, Joe will move to the kitchen.




