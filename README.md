# tallytime

Time tracker based upon the principle that you can only do one thing at a
time.

## Design

The core design of tallytime is that you can only do one thing at a time.

Initially, the output just printed the amount of time spent between the
tasks and it was up to you to add things up.

Since that time, however, I've added support for "tags". Tags are prefixed
by punctuation (any punctuation) and are totaled for you in the reports.

## How I use it

Over the course of the day, I log when what I'm doing changes.

    tallytime %work
    tallytime %work .email
    tallytime %work .cache
    tallytime %break +Lunch
    tallytime %work .cache
    tallytime %break
    tallytime %break +Dinner
    tallytime %work .cache
    tallytime %break +Chatter
    tallytime %work
    tallytime end --tally

When I run "--tally" it produces a report on screen (and in the reports file):

    Mark Sections:
      %break: 10.6883 hours / 641.3000 min
      %work: 8.3269 hours / 499.6167 min
      +Chatter: 6.2986 hours / 377.9167 min
      +Dinner: 2.7200 hours / 163.2000 min
      +Lunch: 1.6669 hours / 100.0167 min
      .cache: 4.4889 hours / 269.3333 min
      .email: 0.3142 hours / 18.8500 min

I originally wrote this tool when I was working a side job and needed to bill
my hours. However, once I had kids I found this was very useful to track
time when family responsibilities interfere with my normal work day.

Using different prefixes it is possible to simultaneously track work using
multiple levels of granularity.

## Usage


    Usage: tallytime [options] time log message

    A stupidly simple time tracker. Supports concurrent marked segments using
    leading punctuation on a word ('started @work on %task'), comments with a
    stand-alone leading punctuation in the comment-line ('% comment only seen in
    raw segment').

    Options:
      --version             show program's version number and exit
      -h, --help            show this help message and exit
      -v, --verbose         display output [default]
      -q, --quiet           be less verbose
      --draft               dump a draft of the report to stdout
      --tally               Instead of logging a time entry, generate the report.
      --last                Print the last-entered log and current time elapsed.
      --home=DIR            Set the default configuration directory. [default:
                        ~/.tallytime
      -s SEGMENTS, --segments=SEGMENTS
                            one or more output segments: raw, comment, mark, secs,
                            last, all, none, (and any can be preceded with 'no' to
                            disable) [default: raw,mark,last]
      --undo                Undo the previous tally.
      --fake=TIMESPEC       Produce timelog output for
                            {YYYY-{MM-{DD'T'}}}HH{:MM{:SS}}
      --write-config        Write the current configuration to the configuration
                            file.

    The time log message can be a single argument or can be multiple arguments.
    There is no support for an on-going task. You start one task, then start a new
    task. The time associated with marks is tracked by summing each of the log
    entries with the same mark. You can have multiple marks in the same log entry,
    and this is the only supported way to track multiple concurrent tasks.


### The different types of segments

<dl>

<dt>none</dt>
<dd>
Disable all segments. Usefully combined with another segment to skip your
defaults and only print that segment.
</dd>

<dt>last</dt>
<dd>Print information about the last entry entered.

<pre>
Last entry: [2016-03-03 13:20:33] %work / 2.3833 min
</pre>
</dd>

<dt>rawraw</dt>

<dd>
This is the "really" raw output.

During the day entries are added to the "log filename" (`~/.tallytime/time.log`
by default) We track time in seconds, but show human-readable times as well.
Using this format in the report file allows you to easily revert a "tallied"
day if you have tallied accidentally.

<pre>
- - - DRAFT MODE - - - - DRAFT MODE - - -
Starting time: 2016-03-03 07:15:58
1457007358 2016-03-03 07:15:58 - %work
1457008102 2016-03-03 07:28:21 - %break
1457016337 2016-03-03 09:45:37 - %work
1457028190 - - - -
</pre>

</dd>

<dt>secs</dt>

<dd>This may currently be broken. It is supposed to allow enabling
(and disabling) view from one of the other segments.
</dd>

<dt>comment</dt>

<dd>
There's a notion of "comments" in the report. These are entries which have a
leading stand-alone punctuation.

This segment (like most segments) ignores the comments.

However, marked tags can exist on a line with a longer description of the work.
This option tracks the full text you entered, and not just the marked tag.

<pre>
Comment Sections:
  %break: 2.2875 hours / 137.2500 min
  %work: 3.6036 hours / 216.2167 min
  %work this is fun: 0.0028 hours / 0.1667 min
</pre>
</dd>

<dt>raw</dt>
<dd>

This mode is closest to the original output of the script.

It is "raw" as in "uncooked".

<pre>
- - - DRAFT MODE - - - - DRAFT MODE - - -
Starting time: 2016-03-03 07:15:58
0.0000 min [2016-03-03 07:15:58] %work
12.4000 min [2016-03-03 07:28:21] %break
137.2500 min [2016-03-03 09:45:37] %work
195.2000 min [- -] -
Last entry: [2016-03-03 09:45:37] %work / 195.2000 min
Time total: 5.7475 hours (344.8500 min)
</pre>
</dd>


<dt>all</dt>
<dd>
Print all segments. This can be combined with disabling specific segments
via the "no" prefix.

    tallytime --draft all,noraw,norawraw

Note that only one raw mode is normally shown and there is a preference for
the "rawraw" mode.

<pre>
- - - DRAFT MODE - - - - DRAFT MODE - - -
Starting time: 2016-03-03 07:15:58
Last entry: [2016-03-03 13:20:33] %work / 0.5833 min

Comment Sections:
  %break: 2.2875 hours / 137.2500 min
  %work: 3.6036 hours / 216.2167 min
  %work this is fun: 0.1019 hours / 6.1167 min

Mark Sections:
  %break: 2.2875 hours / 137.2500 min
  %work: 3.7056 hours / 222.3333 min
</pre>
</dd>


</dd>
</dl>

## Configuration

Here's my current configuration file.

<pre>
[tallytime]
log_filename = time.log
report_filename = report.txt
segments = none,last

[report]
segments = mark,rawraw

[draft]
segments = mark

[log]
segments = 
</pre>

## Correcting or lying about time

We all sometimes forget to log our time. When we're tracking the time for
ourselves, that matters very little.

Since this is based upon plain-text files, it is easy to edit.

There's a "--fake" command-line argument to help you create an entry
that is correctly formatted in the time.log file. If gets out-of-order with
the surrounding entries, you'll need to manually edit the file to correct it,
as it is simply appended to the end of the time.

The "--fake" option takes a timestamp. Use an ISO date-time without timezone.
You can either use the human-friendly ":", " " and "-" separators or the
computer-friendly "", "T" and "" separators. (Just remember that a single
two-digit number is an hour, not a day. To get dates parsed, you'll always
need to specify at least an hour.)

## Accidental --tally and --undo

I was logging additional information in my reports so that when I accidentally
hit --tally, I could copy the data back to the in-progress file and pick up
where I left off. This was sub-optimal and prevented me from getting the
report formats that I wanted, so it was fixed in the 1.0 release.

We now store the last-tallied `time.log` file, and copy it back through
the `--undo` command.

The command won't work if you've created a new `time.log` file, but you'll still
be able to copy the data from the `time.undo` file to the `time.log` file.

