---
date: 2018-10-28T23:00:00Z
title: The Thue-Morse Sequence
url: /thue-morse/
---

{{< noescape >}}
<style>
blockquote p:not(:first-child) {
  display: block;
}
</style>
{{< /noescape >}}


# Making things even

When I was little I liked (had a compulsion[^compulsion]) to make certain things symmetrically even: if I accidentally brushed my face with my left hand, I might purposefully brush it with my right.

Sometimes left then right wasn't enough: the right touch had been more recent! As the mental weight of each touch decayed with time, the right touch remained always a little stronger. The fix to this inequity was to tap, brush, or nibble again; this time first right, then left.

Left, then Right; then Right, then Left.

LRRL

But that wasn't quite even either! Repeating the opposite pattern helped a bit:

LRRL **RLLR**

And to address the unfairness of that pattern:

LRRL RLLR **RLLR LRRL**

and so on. When I try tapping out this sequence on my desk now I can double the length two more times before having trouble and I think I used to take it further.

Years after I stopped doing this I discovered an entry in the [Online Encyclopedia of Integer Sequences](https://oeis.org/A010060) by typing it out as ones and zeros: it was the Thue-Morse Sequence! I remember being pretty excited that the thing I had come up with on my own was shared (and a wee bit disappointed that I didn't get to publish it myself).

---

More recently it occurred to me to Google for the sequence. [Sriharsha Bangaru](https://twitter.com/pfatagaga)'s [writeup](http://linkdot.link/thue-morse-prouhet-sequence.html) is the best thing I found, covering lots of interesting bits: formalisms, connection to the Koch curve, and its application to equal partitioning. But I found a lot of other posts too.  It turns out a lot of people know, and I think have independently discovered, this sequence.

The comments sections of [this LiveJournal post](https://trapezzoid.livejournal.com/82863.html) and [this fun Youtube video](https://www.youtube.com/watch?v=prh72BLNjIk) by [Matt Parker](http://standupmaths.com/) have more reactions like "oh that thing, I thought it was just me!" Here are a few mentions from around the web.

<a href="https://www.youtube.com/watch?v=prh72BLNjIk">
<img src="/assets/abba.png" style="width:100%;"/>
</a>

<a href="https://trapezzoid.livejournal.com/82863.html">
<img src="/assets/abba10.png" style="width:100%;"/>
</a>

<a href="https://www.reddit.com/r/AskReddit/comments/4sx671/do_you_have_a_strange_habit/">
<img src="/assets/abba3.png" style="width:100%;"/>
</a>

<a href="https://www.reddit.com/r/AskReddit/comments/5tywb5/what_is_one_of_your_weird_quirks/">
<img src="/assets/abba6.png" style="width:100%;"/>
</a>

<a href="http://www.ourhealth.com/conditions/nerve-conditions/i-have-tics-but-i-dont-think-its-tourettes-please-help">
<img src="/assets/abba2.png" style="width:100%;"/>
</a>

<a href="https://www.reddit.com/r/OCD/comments/3s85xq/i_think_i_kind_of_have_ocd_about_having_ocd_just/">
<img src="/assets/abba4.png" style="width:100%;"/>
</a>

<a href="https://boardgamegeek.com/thread/1275055/whats-oldest-codecombination-you-know#body_article17560325">
<img src="/assets/abba1.png" style="width:100%;"/>
</a>

<a href="https://www.reddit.com/r/Minecraft/comments/143nga/new_torch_pattern_made_with_ocd_people_in_mind/">
<img src="/assets/abba5.png" style="width:100%;"/>
</a>

<a href="https://www.reddit.com/r/AskReddit/comments/103x2j/what_superstitious_behaviors_do_you_engage_in/">
<img src="/assets/abba7.png" style="width:100%;"/>
</a>

<a href="https://www.reddit.com/r/AdviceAnimals/comments/25fwll/its_an_automatic_reaction_now/">
<img src="/assets/abba8.png" style="width:100%;"/>
</a>

{{< tweet 864546749523910659 >}}
{{< tweet 528791584486457344 >}}
{{< tweet 872822703610359809 >}}

# So what?

So I share a mental quirk with some other people. It's pretty cool that a bunch of humans felt something and wrote it down so they could share it on the internet, but that doesn't distinguish it from the rest of the human canon about feelings we share like "love" and "ennui" and "don't you hate it when the toast falls butter side-down."

It's the terrific specificity of this shared experience that makes it special. Finding out you share knowledge of a (literally and figuratively) transcendental number you discovered on your own feels more dramatic than finding out other people also have [dreams about their teeth falling out](https://www.frontiersin.org/articles/10.3389/fpsyg.2018.01812/full). I know more terms in this sequence than I do digits of [pi](https://oeis.org/A000796) or [e](https://oeis.org/A001113). To this day, it feels like my sequence -- I taught an embedded device to blink it as a Hello World program last month -- but also like a universal truth of the universe.

It's cool to know something that felt so intensely personal, so singular, is shared by others. It's like knowing a secret handshake that no one taught you. Like being a mutant and discovering the X-Men, or a wizard raised by muggles discovering Hogwarts, it's a small piece of that kind of happiness of not being alone. Thanks internet :)

[^compulsion]: A compulsion is the inexplicable need to perform a repetitive behavior. An obsession is a recurring thought or fear, which might be about what could happen if a compulsion is not observed. For a diagnosis of Obsessive-Compulsive Disorder you look for both. I worked at a psychiatry lab on the imaging side for long enough to know I'm not qualified to say much about a diagnosis like [OCD](https://www.youtube.com/watch?v=Ug19JBIIycs), but it seems important to mention it because many of the people who post online about this sequence identify this way. If you, like I did as a child, feel a compulsion for symmetry, this does not mean you have OCD.
