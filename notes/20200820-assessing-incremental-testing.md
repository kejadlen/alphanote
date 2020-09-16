# Assessing Incremental Testing Practices and Their Impact on Project Outcomes

This research looked at ~150 undergrads working on multiple projects throughout
a course (with ~1k LoC each), finding that there's a correlation between
testing effort and correctness (unsurprisingly) and a negative correlation
between testing first and code coverage (!).

There are the usual caveats of extrapolating from college students to a
professional workforce, and (fully admitting to No-True-Scotsman-ing here) I'm
skeptical that the students were actually using TDD rather than just
testing-first. (Yes, there is a difference.)

Kind of neat: they used an Eclipse plugin that just committed on each save to
get an event stream of student code editing.
