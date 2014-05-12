---
title: Fizz-Buzz by Exhaustion in Ruby
kind: article
category: code
created_at: 2013-03-27
category: article
---

<!-- _. -->

Here's a cute solution to [Fizz-Buzz](https://en.wikipedia.org/wiki/Bizz_buzz)
that occurred to me when I read about the
`unless` statement in Ruby; it uses only the unless statement.
I'm doing the Bitmaker Labs workshop, and one of the questions for the
interview was Fizz-Buzz.

I call it "by exhaustion" because, instead of using `if`-statements to look for
which terms to include, it uses `unless`-statements to look for which terms
to exclude (until all possibilities are exhausted, in which case it just
gives up and says "Let's just assume it's 'Buzz'").

    (1..100).each do |n|
      (puts n.to_s; next) unless (n % 3 == 0) or (n % 5 == 0)
      (puts "Fizz"; next) unless (n % 5 == 0) or (n % 3 != 0)
      (puts "FizzBuzz"; next) unless (n % 3 != 0)
      puts "Buzz"
    end
{: class="language-ruby" }
