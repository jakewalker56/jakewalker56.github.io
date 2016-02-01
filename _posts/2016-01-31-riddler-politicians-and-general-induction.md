---
layout: post
title: The Riddler, Politicians, and General Induction Problems
date: 2016-01-31
---

The following question comes to us from 538's [The Riddler](http://fivethirtyeight.com/tag/the-riddler/), a wonderful weekly column written by [Ollie Roeder](https://twitter.com/ollie).

>Suppose that five politicians, disgusted with the current two-party system, come together to choose a third-party candidate to run in the 2016 presidential election. The politicians’ names are Anders (A), Blinton (B), Cubio (C), Drump (D) and Eruz (E). Not wanting to spend all their time campaigning in Iowa and New Hampshire in winter, they decide instead to pick which of them will be the candidate at a secret meeting with just the five of them. The voting procedure is as follows: They will first hold a vote of A versus B. (The five politicians are the only voters.) The winner of that vote will then be paired against C. That winner will be paired against D, and finally that winner will be paired against E. They will declare the winner of that last matchup to be their candidate.

>Each of A, B, C, D and E wants to be the presidential candidate themselves, but also has clear preferences over the others. Furthermore, the politicians’ preferences are common knowledge. Their preferences are as follows (“X > Y” means Candidate X is preferred to Candidate Y):
>Candidate A: A > B > C > D > E
>Candidate B: B > A > E > D > C
>Candidate C: C > D > A > E > B
>Candidate D: D > B > A > E > C
>Candidate E: E > D > B > C > A
>All of the politicians are forward-looking and vote strategically.
>Question 1: Who will be chosen as the presidential candidate?


Brain teaser wonks may recognize this as a variant of the [Pirate Game](https://en.wikipedia.org/wiki/Pirate_game) or the [Survival of the Sheep](http://www.braingle.com/brainteasers/9026/survival-of-the-sheep.html), but more generally it's a question of induction.  

In order to answer the question "What will the politicians vote for in this round?" you must first answer the question "what happens in the next round if the vote isn't decided in this round?"  But in order to answer that question, you will have to look at what happens in the round-after-next, and then the round-after-the-round-after-next, etc.  To solve the problem, you must start in the last round, and work your way forward.

So if we make it to the last round, what will happen?  We simply ask which candidates prefer D over E, and which prefer E over D?

Last Round Voting (4th round)
---

Candidate | Vote's for...
:-:|:-:
A | D
B | E
C | D
D | D
E | E


So we see that if we make it to the last round, candidate D will win.

Now let's look at the second to last round (C vs. [Winner of next round]).  If C doesn't win, we know that D will win.  So now we look again, and we ask ourselves which candidates prefer C over D?

Second-to-Last Round Voting (3rd round)
---

Candidate | Vote's for...
:-:|:-:
A | C
B | D
C | C
D | D
E | D

So once again, D wins!  We can repeat this process for the previous rounds as well:

Third-to-Last Round Voting (2nd round)
---

Candidate | Vote's for...
:-:|:-:
A | B
B | B
C | D
D | D
E | D


Fourth-to-Last Round Voting (1st round)
---

Candidate | Vote's for...
:-:|:-:
A | A
B | A
C | D
D | D
E | D
 

Hence, we see that D is going to win it all.  In the first round, candidate's C, D, and E all prefer the outcome of the second round to voting for A, so they will all vote for B.  In the second round, candidates C, D, and E all prefer the outcome of the third round to voting for B, so they will all vote for C.  In the third round, candidates B, D, and E all prefer the outcome of the 4th round over voting for C, so they will all vote for D.  In the fourth and final round, candidates A, C, and D all prefer candidate D to candidate E, so they will vote for D, and D takes it.

This turns out to be a fairly easily generalizable problem.  All we really need are the order of voting rounds, and a list of candidate preferences, and we can simply walk backwards through the rounds and figure out optimal behavior.  That code (in Ruby) looks like this:

```ruby
def simulate(preferences, vote_order)
	winners_by_round = []
	num_rounds = vote_order.length - 1
	num_candidates = preferences.length

	num_rounds.downto(1).each do |i|
		matchup_a = vote_order[i - 1]
		matchup_b = vote_order[i]

		vote = []
		if winners_by_round.length == 0
			next_winner = matchup_b
		else
			next_winner = winners_by_round[0]
		end 

		0.upto(num_candidates - 1).each do |j|
			if preferences[j].index(matchup_a) < preferences[j].index(next_winner)
				vote[j] = 0
			else
				vote[j] = 1
			end
		end
		if vote.inject(:+) <= num_candidates.to_f / 2.0
			winners_by_round.unshift(matchup_a)
		else
			winners_by_round.unshift(next_winner)
		end
	end
	return winners_by_round
end
```

Note that the `winners_by_round[i]` value answers the question "given that we made it to round i, who is going to win?"  You can find the fully-commented code [here](https://github.com/jakewalker56/ruby-scripts/blob/master/induction_solver.rb). 

Now to solve our specific version of the problem, we call our simulate function with the following arguments:

``` ruby
preferences = []
#Candidate A: A > B > C > D > E
preferences << [0, 1, 2, 3, 4]
#Candidate B: B > A > E > D > C
preferences << [1, 0, 4, 3, 2]
#Candidate C: C > D > A > E > B
preferences << [2, 3, 0, 4, 1]
#Candidate D: D > B > A > E > C
preferences << [3, 1, 0, 4, 2]
#Candidate E: E > D > B > C > A
preferences << [4, 3, 1, 2, 0]

#vote_order specifies the order the votes happen in.  [0, 1, 2] would mean that everyone votes 
#between 0 and 1, and the winner is then voted against 2, etc.

vote_order = [0, 1, 2, 3, 4]

#Question 1: Who will be chosen as the presidential candidate?
q1_winner = simulate(preferences, vote_order, debug)[0]
```

Confirmed: D is our winner.  Now let's take a crack at part 2!


>Now assume that A has the flu and is forced to miss the voting meeting. He is allowed to transfer his vote to someone else, but he can’t make that other person commit to vote against her own self-interest.

>Question 2: To whom should he transfer his vote, given his candidate preference outlined above (A > B > C > D > E)?

>Question 3: Who will win the candidacy now?

>Question 4: A month before the meeting, Candidate A must decide whether or not to get the flu vaccine. Should he get it?


In this twist of the problem, we ask ourselves the question "What happens if we replace A's utility function with someone else's?  Who's should A use to maximize their payoff?"

In this case, we can simply loop through everyone else's preferences and see which one yields the best result:

```ruby
#worst thing that can happen is A's least favorite wins
q2_winner_array = [preferences[0].last]
q2_transfer = 0
1.upto(preferences.length-1).each do |i|
	temp_preferences = preferences.dup
	temp_preferences[0] = temp_preferences[i]
	temp_winner_array = simulate(temp_preferences, vote_order)
	if preferences[0].index(temp_winner_array[0]) < preferences[0].index(q2_winner_array[0])
		#if A likes the current winner more than any other he's seen so far...
		q2_transfer = i
		q2_winner_array = temp_winner_array 
	end
end
```

After running this script, we find that by replacing his preferences with those from candidate E, candidate A can force himself to become the new winner!  This happens because given the new preferences, E will win if we make it to round 2, and the other three candidates all prefer A over E, so they vote for A.

This is a counterintuitive outcome.  This tells us that changing your utility function changes the outcome, sometimes for the better according to your old utility function.  That is to say, sometimes you can't get what you want, but you would get it if only you wanted something else.

The trick to all this is that it relies on that perfect information assumption stated above.  Everyone else's actions are causally influenced by your preferences, so you can end up with weird results like this.  This tells us that if you can FAKE a specific voting preference, you can manipluate everyone else into giving you what you want.  What we have here is mathematical proof that lying totally works.

Note here that the voting order matters for question 2-4. If we switch up the voting order as below, we get a different result:

```ruby
preferences = []
preferences << [0, 1, 2, 3, 4]
preferences << [1, 0, 4, 3, 2]
preferences << [2, 3, 0, 4, 1]
preferences << [3, 1, 0, 4, 2]
preferences << [4, 3, 1, 2, 0]

######## Here's the big change!  #########
vote_order = [4, 3, 2, 1, 0]
##########################################

q2_winner_array = [preferences[0].last]
q2_transfer = 0
1.upto(preferences.length-1).each do |i|
	temp_preferences = preferences.dup
	temp_preferences[0] = temp_preferences[i]
	temp_winner_array = simulate(temp_preferences, vote_order)
	if preferences[0].index(temp_winner_array[0]) < preferences[0].index(q2_winner_array[0])
		#if A likes the current winner more than any other he's seen so far...
		q2_transfer = i
		q2_winner_array = temp_winner_array 
	end
end
```

In this world, A would instead tranfer their vote to either C or D, and D would still end up winning.

Extending the Question
=====

So if order matters, you might wonder whether you would prefer to be the first voted on, or the last?  Going last in this context is fundamentally different than getting a "buy week" in sports, because the whole system is deterministic.  There's no "percentage chance of losing" in the first round, so it's not obvious whether you'd prefer to be first, last, or in the middle.

To investigate this question, we'll simulate randomly generated preferences with different numbers of candidates, and we'll see who ends up winning the most often (we assume everyone always prefers themselves).  We'll plot this to a file using [gnuplot](https://github.com/rdp/ruby_gnuplot):

{% highlight ruby %}
Gnuplot.open do |gp|
  Gnuplot::Plot.new( gp ) do |plot|
  
	candidate_trials = [5, 10, 25, 100]
    title_string = candidate_trials.join("_")

    plot.terminal "png"
    plot.output File.expand_path("../" + title_string + "_win_percentage.png", __FILE__)
    plot.xrange "[0:9]"
    plot.title  "Win Percentage by Vote Order"
    plot.ylabel "Win Percentage"
    plot.xlabel "Order voted on (0 = first vote, 1 = second vote, etc.)"
    
    num_points = 10
    plot.data = []

    for k in candidate_trials
		num_candidates = k
		num_simulations = 10000
		winner_array = []
		vote_order = (0..(num_candidates-1)).to_a
		num_simulations.times do
			preferences = []
			num_candidates.times do |j|
				preferences[j] = (0..(num_candidates-1)).to_a.shuffle
				#remove himself from preferences
				preferences[j].delete(j)
				#add himself to front- everyone prefers themselves!
				preferences[j].unshift(j)
			end
			winner_array << simulate(preferences, vote_order, debug)[0]
		end
		percent_winners = []
		num_candidates.times do |i|
			percent_winners[i] = winner_array.select{|v| v == i}.count.to_f / num_simulations.to_f
		end
	    x = (0..(num_points-1)).to_a
	    y = percent_winners.first(num_points)
	      
	    plot.data << Gnuplot::DataSet.new( [x, y] ) { |ds|
	        ds.with = "linespoints"
	        ds.title = num_candidates.to_s + " Candidates"
	      }
	end

  end
end
{% endhighlight %}

This yields the following graph:

![Win Percentage as a function of Voting Location]({{ site.url }}/assets/5_10_25_100_win_percentage.png)

So a few interesting things here.  First, it's hugely advantageous to go first.  In the case where there's 100 candidates, you have a 20% chance of winning when you go first.  Going last give you approximately a 0% chance.

Second, there's some weirdness going on with regard to even and odd numbers of candidates.  You might wonder why going first is so much better with 10 candidates than with 5 candidates, but also much better with 10 candidates than with 100 candidates.  In other words, why do these lines cross at all?  Let's examine further by plotting 5-10 candidates on their own graph:

![Win Percentage as a function of Voting Location, smaller candidate pools]({{ site.url }}/assets/5_6_7_8_9_10_win_percentage.png)

So that's super weird, right?  It seems like you get an even larger comparative advantage by going first if there is an even number of candidates vs. an odd number of candidates.  We suspect this is perhaps because our implementation has ties going to the "stop the votes, we have a winner" side.  If we change our `<=` to just `<`, we get the following:

![Win Percentage as a function of Voting Location, smaller candidate pools, different tiebreaker]({{ site.url }}/assets/5_6_7_8_9_10_win_percentage_tiebreak.png)

And here we see a complete reversal!  Candidates get a significantly higher advantage by going first if there are an odd number of candidates than even.  Not only that, but we see big jumps in win percentage at the tail end of the even candidate vote order.  This happens because for small numbers of candidates, we're going to see a whole lot of ties happen.  If nobody ever gets better than a tie, and we're down to the last two people, we break the tie by going with the person who's at the end of the voting queue.  

We expect that kick at the end to get smaller and smaller as the number of candidates gets larger, because we are less likely to get to the end with a tie.  We check what happens for the n=100 case:

![Win Percentage as a function of Voting Location, large number of candidates, different tiebreaker]({{ site.url }}/assets/100_win_percentage_tiebreak.png)

Science!



