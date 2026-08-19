# A city library, modelled

This repository is a worked example. It contains one complete model of a city
library — what the library owns, who borrows it, what happens when a book comes
back late — written down in [ESDM](https://www.esdm.io/), a notation for
describing a domain as a set of plain YAML documents.

Nothing here runs. There is no library management system in this repository, no
database, no server. The model is documentation with a strict enough structure
that tools can read it, check it, and draw it. The point is to be read.

The idea of using a city library to teach event sourcing is not ours. It comes
from [the native web](https://www.thenativeweb.io/), the people who build ESDM
itself and who have been teaching event sourcing and domain-driven design with
this example for years. What follows is one worked-out version of their idea;
the credit for the idea belongs to them.

## Why it exists

Most modelling examples fail in one of two directions. Either they are toys — one
entity, three fields, a single command — and collapse the moment you ask a real
question of them. Or they are real systems, where the domain itself takes a week
to understand and the modelling lessons are buried under it.

A city library sits between the two. Everyone has been in one, so nobody needs
the domain explained. And yet it has genuine modelling problems, the same ones
that show up in insurance, logistics and banking:

- The same word means different things to different people.
- Some rules can't be checked by looking at one record.
- Things happen because time passed, not because anyone did anything.
- The thing you write and the thing you read are not the same shape.

So the library is the vehicle, not the subject. If you want to learn how to model
a domain — how to find the boundaries, where to put a rule, when something
deserves to be its own concept — this is a model you can read end to end in an
afternoon and argue with.

## What it teaches

**The same word means different things in different places.** In the catalog, a
"book" is a title — one entry, one ISBN, however many physical volumes sit on the
shelf. In lending, a "book" is one of those volumes; a title with three copies is
one book to the cataloguer and three to the person at the desk. Rather than
inventing a compromise word that satisfies neither, the model keeps two separate
areas, each with its own vocabulary, and is explicit about which word is banned
in which. This is the single most useful idea in domain-driven design, and it is
easiest to see when the two meanings are this close together.

**Not every part of the business deserves the same attention.** Lending is what a
library is *for* — it's where the model goes deep. Cataloguing is necessary but
unremarkable; every library does it the same way. Library-card administration you
would simply buy. The model says so out loud, and that classification is what
justifies spending effort in one place and not another.

**State is something that happened, not something you edited.** A copy isn't "on
loan" because a field somewhere says so. It's on loan because it was borrowed and
hasn't come back yet. The model records the facts — borrowed, returned, reminded,
fee incurred — and everything else is derived from them. That's event sourcing,
and the practical consequence is that you can always answer "how did it get like
this?"

**Writing and reading are different problems.** The rule "you can't borrow a copy
that's already out" has to be enforced in exactly one place, at the moment
someone tries. "What's available right now" is a different question with a
different shape, and the model builds purpose-made views to answer it — including
two different views over the very same facts, a flat list for staff and a search
index for visitors. Separating the two is what CQRS means; seeing two views over
one event is what makes it click.

**Some rules don't live inside a single thing.** "A borrower may hold at most
three reservations" is not a property of any one reservation. The model shows two
different ways of handling that: the familiar one, where a real thing owns the
rule (the copy on the hold shelf), and the less familiar one, where the boundary
is drawn fresh for each decision by asking which past events matter. Both appear
in the same area of the model, side by side, so the trade-off is visible rather
than asserted.

**Time is a participant.** A copy falls due. A week later, if it hasn't come back,
a reminder goes out. After three reminders the library stops asking and the card
is blocked. Nobody triggers any of this; it happens because dates passed. That
needs something that waits, remembers what it already did, and knows when to stop
— and the model shows what that looks like, including how it ends.

**Reacting to something comes in two flavours.** Telling the world — mailing the
borrower their due date — is different from doing the next thing in the domain —
putting a returned copy on the hold shelf for whoever was waiting. The model keeps
them as separate kinds of thing and never mixes the two in one place, because the
moment you do, you can no longer tell which of your reactions change your
business and which merely narrate it.

**Dependencies between areas have names.** Reservations can't fulfil its promise
alone; it needs lending to actually hand a copy over. The self-service kiosk speaks
the terminal vendor's protocol, and that protocol is translated at the door rather
than allowed inside. The card system was bought, so its model is taken as given
rather than negotiated. Each of these is a different, named relationship, and
naming it is what turns "these two things talk to each other" into a decision you
can review.

**You can write down what should happen before it happens.** For each significant
rule the model carries concrete worked examples: given these things happened, when
someone does this, then that follows. They read like test cases, but they exist
here as specifications — the place where "a copy that's already out can't be
borrowed" stops being a sentence and becomes something checkable.

**Modelling starts by listening.** Before any of the structure existed, someone
described how borrowing actually works today, in the librarian's own words. That
description is still in the repository, as is a second one imagining how
collecting a reserved book *should* feel, told without reference to any software at
all. The model was read out of the first; the second is what the model is aiming
at. Keeping both is more honest than keeping either.

**What you leave out is also a decision.** See the last section.

## A tour of the model

The library is divided into four areas, each with its own vocabulary and its own
rules. Three of them mirror how a library actually thinks about itself; the fourth
is the paperwork.

### Cataloguing — what the library owns

The narrowest area, and deliberately so. A book here is a title: author, title,
ISBN. It gets acquired, which is the moment it enters the catalogue, and that's
essentially the whole lifecycle — cataloguing never counts volumes and never cares
who has one.

What it feeds: two ways of looking at the catalogue. Staff get a plain list. The
public gets free-text search over the same acquisitions, ranked by relevance — a
different tool for a different question, built from identical facts.

### Lending — who has what, and until when

The heart of the model, and the part a library couldn't buy off the shelf.

A copy is one physical volume. It gets put on the shelf, borrowed, returned,
reminded about. Two rules hold it together: a copy that's out can't be borrowed
again — the second request is refused, not queued — and a copy has a borrower and
a due date exactly while it's out, never otherwise.

Bring one back late and you owe a fine. The first three days are free, because the
library would rather have the book back than the money, and the fine never exceeds
what replacing the copy would cost. This is the one place in the model where a
single action produces two different facts, and only sometimes: returning a book
always records the return, and records a fee only if it was actually late.

What it feeds: what's on the shelf right now, and what a borrower currently owes.

### Reservations — waiting for a copy, and the hold shelf

Reserving is claiming the next available copy of a title. You may have three
reservations at once, and not two for the same title.

Then the copy comes back, and it becomes a hold: pulled off the shelf, a slip with
your name tucked into it, waiting at the desk. This is the part most reservation
systems get wrong by treating it as the same thing — the model treats it as a
distinct, later state, and is explicit that "hold" is the wrong word for a
reservation precisely *because* it's the right word for this.

This area is where the two ways of enforcing a rule sit next to each other. The
hold is a thing that exists and owns its own rules. The reservation limit isn't a
thing at all — it's a question asked fresh each time somebody reserves, answered by
looking at what that borrower has already reserved, released and collected.

### Membership — library cards

The paperwork, and the part you would buy rather than build. A card gets issued and
can be blocked. That's it.

It exists for two reasons. Everywhere else in the model a borrower is just a card
number, with a note saying identity lives elsewhere — this is that elsewhere, made
concrete. And persistent overdue now has a consequence: after the third ignored
reminder, the card stops working.

### The seams between them

The most interesting documents in the model are the ones that don't belong to any
single area:

- A newly catalogued title becomes lendable without anyone re-typing it.
- Borrowing a copy sends the borrower its due date.
- A copy falling due starts the reminder sequence; the copy coming back stops it.
- A returned copy that somebody is waiting for goes onto the hold shelf for them —
  oldest reservation first — rather than back into circulation.
- A hold being placed tells the borrower their book is ready.
- The third ignored reminder blocks the card.

Plus two things outside the library entirely: whatever actually delivers mail, and
the self-service terminal in the foyer, whose firmware speaks its own language and
gets translated at the boundary.

### The two stories

`taking-a-copy-home` records how borrowing works today, coarsely, including the
software that's already involved — the librarian's line was that nothing counts
until the scanner beeps.

`collecting-a-hold` imagines how collecting a reserved book should go, in fine
detail, with no software in it at all. It's the front side of the hold shelf, told
the way a workshop actually starts.

### The worked examples

Four sets, one for each kind of rule in the model: borrowing a copy, the
reservation limit, the overdue chase, and the availability view. Between them they
pin down the refusals, the empty outcomes, the conditional fee, the timers firing
and being cancelled, and what a borrower actually sees.

## Looking around

The `esdm` command-line tool reads this model. It isn't part of this repository,
and there's nothing to build: it ships as a pre-built binary for macOS, Linux and
Windows, and the download links are listed in the ESDM documentation under
[installing ESDM](https://www.esdm.io/getting-started/installing-esdm/). On an
Apple-silicon Mac, getting it into the repository root comes down to:

    curl -LO https://esdm.s3.fr-par.scw.cloud/0.14.0/esdm-darwin-arm64
    mv esdm-darwin-arm64 esdm
    xattr -d com.apple.quarantine esdm
    chmod a+x esdm
    ./esdm version

Swap `darwin-arm64` for `darwin-amd64`, `linux-arm64`, `linux-amd64` or one of
the Windows builds as your machine requires — the installation page lists them
all, and carries the current version number. Linux skips the `xattr` line;
Windows uses `Unblock-File` instead. This model was written against v0.14.0.

The binary is deliberately ignored by git, so it can sit in the repository root
without ever being committed. From there:

    ./esdm view -d model        # the whole model as a tree
    ./esdm glossary -d model    # the vocabulary, area by area
    ./esdm lint -d model        # check that it all hangs together

The last one prints nothing when the model is sound, and should stay that way — it
catches the errors that matter here, like a rule referring to something that
doesn't exist, or two areas talking to each other without saying how.

If you'd rather just read: start with `domain.esdm.yaml` and `subdomains.esdm.yaml`
for the shape, then take the areas in the order above, then the seams in
`integration/`. Every document explains in prose *why* it's built the way it is,
not just what it contains — that commentary is the actual teaching material, and
the structure is scaffolding for it.

Anything you can't follow without knowing how libraries work on the inside is a
defect in the model, not a gap in your knowledge. That standard matters more here
than covering every feature of the notation.

## What it deliberately leaves out

A good example shows what the domain needs, not everything the notation can
express. Left out on purpose:

- **Money, beyond late fees.** No acquisition budgets, no procurement, no payment
  handling. It would drag purchasing into a model about books and loans.
- **Inter-library loan.** Real, but nothing else here would talk to it — a
  connection with no traffic teaches nothing.
- **Waiting lists for a specific copy.** A copy that's out is simply refused;
  reserving the title is how you wait for it. Two mechanisms for one need would
  muddle both.
- **Branches, shelving, transfers.** Physical logistics is a whole domain of its
  own and not this one.
- **Logins, permissions, roles.** The model says which kinds of people and which
  automations may do what. How anyone proves who they are is somebody else's
  problem.

And some of the notation's own options go unused because a library has no honest
use for them — a nine-way choice where two are true here, delivery guarantees that
would be wrong for every reaction in this model. Picking one just to demonstrate it
would teach the wrong reflex, which is the opposite of what an example is for.

Nothing on this list turned out to be a mistake while the rest of the model was
written. If a natural home for one ever appears, the right move is to take it and
update this section — not to defend the list.
