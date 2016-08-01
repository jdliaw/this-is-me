---
layout: post
title: Statesman
updated: 2016-07-14 01:03:28
category: tech
---

Looking back, this ticket wasn't super exciting from an engineering viewpoint. It wasn't like some interesting problem to solve where you had to come up with a solution that fits a certain criteria. But it was still a meaty ticket.

The weight of it came from having **to change a really fundamental component of the entire app that many other parts relied on** and was inherent to the functionality of those apps.

Basically, Cory wanted to switch to a different gem for handling our state machines in our models in order to keep the state machine parts completely separate from the model itself and its own logic, so that from the model's POV, it's all abstracted and it's just for better organization.

And this was going to touch on four major models, the bigger of which included order items.

Overall, it wasn't a difficult task. Just kind of tedious.

Implementing the new gem itself was pretty easy, especially because the `order` model had already been converted, so I had that as an example to go off of, and even without it, I could easily do it just by looking at their documentation.

Then the tedious part was removing all references of methods that no longer existed (because transitions were being implemented differently now), and then running test suites and of course breaking a TON (like... a TON), and going through each one and fixing them. And of course, I also had to write my own tests for the state machine. Basically, we just wanted to switch to the new gem and have everything still work as it did with the previous gem.

A slightly more interesting subject came up when we were considering which models actually needed to store a history of their transitions, because for some models, we probably don't really care about that, so why take up unnecessary space, right?

The `order` model which had already been implemented was implemented with its own `transition class` because we definitely wanted to keep a history of those. But for these 5 other models, we decided that we would just have the history default to memory and just keep the last transition that occurred. To do this, as per the documentation, you just don't create & specify its own `transition class`, and it automatically defaults to memory. So we stored it in the `state` columm of each table.

As I was going through specs and prying af to figure out how to fix certain ones whose problems were not as apparent, I found that the state machine wasn't reading the last transition from the `state` column in the table, so often it would just go to the initial state and perform its transitions that way, which meant that all the transitions that weren't supposed to be starting from the inital state were wrong.

So we dug through the gem's source code a bit and found that the state machine reads from a `history` attribute to determine what the current state is and from there validate transition requests and perform transitions. So we added a line in the state machine declaration to save the last transition into the history.

```ruby
def state_machine
 +    @_state_machine ||= begin
 +      @state_machine = OrderStateMachine.new(self)
 +      if state
 +        @state_machine.history << Statesman::Adapters::MemoryTransition.new(state, 0)
 +      end
 +      @state_machine
 +    end
 +  end
```

Basically, I spent a lot of time running & fixing specs, so it didn't turn out to be that interesting after all. Sad.

Also, Cory suggested that I submit our fix for persisting transitions to memory as a PR to the ***Statesman*** gem as a cool open source kind of contribution, which sounded pretty dope to me, so I opened up an issue about it but they haven't said a thing... rip.
