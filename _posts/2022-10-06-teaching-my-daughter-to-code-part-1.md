---
layout: post
title: Teaching my Daughter to Code - Part 1
date: 2022-10-06T12:15:00+00:00
categories:
  - family-time
tags:
  - coding-for-kids
---

# Background

The past couple of days have been a lot of fun.  I've been taking over
getting my five-year-old daughter ready for school in the morning because
my wife has been sick.  I was sitting next to my daughter and she started
to show an interest in what I was doing.  So I showed her.  I explained
the tiny bit of Ansible I was working on, and she wanted to help.  So I
opened up TextEdit on my Mac M1 and let her play.  

Keeping in mind at this point her experience level with the keyboard is
limited to playing minecraft with daddy, and that she doesn't yet have 
the vocabulary to spell much I just sat next to her as she explored the
action bar color pallet and started typing rainbow gibberish.  After a
while, I got the idea of giving her simple three letter words to sound
out and spell while finding the letters on the keyboard and sounding out
the letters for her.  This was extremely successful, and she had a bunch
of fun doing it.  So, today I decided let's do something different and
introduce my daughter to the concept of teaching a computer to do something
useful.

# First step

Open up a terminal and maximize it so that it fills the screen.  This will
hide distractions.  Next, launch python3.  Wait for child to finish getting
ready for school. Then chat with the child to get them excited about telling
the computer to do things.

My daughter is a fan of the Frozen franchise.  Her favorite character is Elsa.
So, when I asked her what she wanted to make the computer print, her first
thought was, "iris and elsa".  So, I guided her character-by-character how to
instruct the computer in python to print "iris and elsa".

```shell
>>> print('iris and elsa')
iris and elsa
```

The next step was to have her tell the computer to do it again. So we did it
agaion a few times...

```shell
>>> print('iris and elsa')
iris and elsa
>>> print('iris and elsa')
iris and elsa
>>> print('iris and elsa')
iris and elsa
```

# Next Step, Tell the Computer to Count Elsas

Next, I said to my daughter, that telling the computer to do the same thing
over and over again seemed like a lot of work.  Maybe we could tell the
computer to do something 5 times, like counting Elsas.  I asked her if she
knew what a loop was, and she said "it's like a circle".  I replied, yes a
loop is a set of things that repeat a set number of times over and over again.
Then I asked, should we write a loop that counts Elsas?  Do you want to print
the number of Elsas before or after her name?  "After" my daughter says
excitedly.  How many Elsa's do we want the computer to count? "Nine" my daughter
says.  Here's the loop we came up with.

```shell
>>> for i in range(1,9):
...   print(f'elsa {i}')
...
elsa 1
elsa 2
elsa 3
elsa 4
elsa 5
elsa 6
elsa 7
elsa 8
```

What happened?  How many Elsas got printed?  "Eight" my daughter says.  But,
we wanted nine Elsas.  What do you think will happen if we put 8 in for the
range?

```shell
>>> for i in range(1,8):
...   print(f'elsa {i}')
...
elsa 1
elsa 2
elsa 3
elsa 4
elsa 5
elsa 6
elsa 7
```

Wow, this time we got 7 Elsas.  If 8 got us 7 Elsas, and 9 got us 8 Elsas,
what number do we need to get 9 Elsas?  "Ten!" my daughter exclaims.

```shell
>>> for i in range(1,10):
...   print(f'elsa {i}')
...
elsa 1
elsa 2
elsa 3
elsa 4
elsa 5
elsa 6
elsa 7
elsa 8
elsa 9
```

"What happens if we use 11?" my daughter asks.

```shell
>>> for i in range(1,11):
...   print(f'elsa {i}')
...
elsa 1
elsa 2
elsa 3
elsa 4
elsa 5
elsa 6
elsa 7
elsa 8
elsa 9
elsa 10
```

"What about 12?" she asks again.

```shell
>>> for i in range(1,12):
...   print(f'elsa {i}')
...
elsa 1
elsa 2
elsa 3
elsa 4
elsa 5
elsa 6
elsa 7
elsa 8
elsa 9
elsa 10
elsa 11
```

# Teaching the Computer to Count Elsas

I tell my daughter, that it seems like a lot of work telling the computer all
of the instructions to count Elsas over and over again.  Maybe we could
teach the computer to count Elsas for us.  Maybe we could make it so that it
will give us exactly the number of Elsas we tell it to.  "Yeah!" my daughter
exclaims.  I explain that a function is how we teach the computer new
commands it can execute.  We decided the name of the function should be
`count_elsas`.  I then explained that we needed a variable `n` to hold the
number of elsas we wanted the computer to count.  We copied our previous function
code and I asked her what the difference was between how many Elsas we told the
computer to print and the number of Elsas that got printed.  Twelve minus eleven
equals?  "One" my daughter exclaims.  I explained how we tell that to the computer.
This is the function we came up with.

```shell
>>> def count_elsas(n):
...   for i in range(1,(n+1)):
...     print(f'elsa {i}')
...
```

What happened?  Why didn't the computer print any Elsas?  "I don't know" my daughter
responds.  I explain that we only told the computer how to count Elsas, we didn't ask
it to count any Elsas.  How many Elsas should we tell the computer to count?
"Twelve!" my daughter exclaims.

```shell
>>> count_elsas(12)
elsa 1
elsa 2
elsa 3
elsa 4
elsa 5
elsa 6
elsa 7
elsa 8
elsa 9
elsa 10
elsa 11
elsa 12
```

# Conclusion

I had a lot of fun this morning spending time with my daughter sharing my passion.
While she isn't able to write any of this code on her own yet, she's been exposed to
the following computer science primatives:

- Output
- Loops
- Functions
- Input

As I was taking her to school this morning, she told me "I want to be a computer guy like
you when I grow up".  I told her that she can definitely become a computer scientist when
she grows up and we booked it to school.