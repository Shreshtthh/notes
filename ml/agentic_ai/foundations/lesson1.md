Back to chapter

Prev
Lesson 1

Variance and Standard Deviation — Why We Square
Lesson 1

Mean and Median — Summarizing with One Number
Pillar: SUMMARIZE — "Replace a list of numbers with a single number that captures the center."

Why Statistics Matters for Modern AI
Every loss function you'll meet in this course — mean squared error, cross-entropy, KL divergence, GRPO advantage — is a statistic. A statistic is a number computed from a list of other numbers, designed to summarize them.

When a neural network is "learning", it is adjusting its parameters so that some statistic of its predictions (the loss) gets smaller. When you read a paper that says "we beat the baseline by 3%", that's a statistic. When you debug a model and look at "average attention to position 0", that's a statistic.

Statistics is not a side topic. It is the language in which every model's behavior is described, every loss is defined, and every result is reported. The next two chapters teach you the small handful of statistical ideas that show up everywhere in deep learning.

We start at the simplest possible question: given a list of numbers, what's a good single number to summarize them?

The Mean
The arithmetic mean — often just called the average, or in probability theory the expected value (the long-run average you would see if you kept drawing samples) — is the sum divided by the count.

x
ˉ
=
x
1
+
x
2
+
⋯
+
x
n
n
=
1
n
∑
i
=
1
n
x
i
x
ˉ
 = 
n
x 
1
​
 +x 
2
​
 +⋯+x 
n
​
 
​
 = 
n
1
​
 ∑ 
i=1
n
​
 x 
i
​
 

Example. Five test scores: 
{
72
,
85
,
90
,
65
,
88
}
{72,85,90,65,88}.

x
ˉ
=
72
+
85
+
90
+
65
+
88
5
=
400
5
=
80
x
ˉ
 = 
5
72+85+90+65+88
​
 = 
5
400
​
 =80

The class averaged 80. Notice that no individual student scored 80 — the mean is a summary, not a fact about any one observation.

Where the Mean Came From
The mean is so familiar that it feels like it was always there. It wasn't.

For most of recorded history, when people had multiple measurements of the same thing — the diameter of a coin, the position of a star, the weight of a sack of grain — they did not average them. They picked the measurement they trusted most. A senior astronomer's observation outranked a junior's. A clearer night's reading beat a hazier one. Combining measurements arithmetically felt like contaminating the good ones with the bad.

The first systematic use of the arithmetic mean for combining noisy measurements came from astronomy in the late 1500s. Tycho Brahe, the Danish astronomer who built the most accurate pre-telescope observatory in Europe, faced a real problem: his sextants and quadrants gave slightly different angles every night, and he had to commit to a single number for each star's position. He started averaging. The catalogue this produced was so much better than anything before it that it later let Kepler discover the elliptical orbits of the planets.

Why did averaging work? The intuition only got formalized two centuries later. Pierre-Simon Laplace (around 1810) showed mathematically that under reasonable noise assumptions, the average of 
n
n independent measurements is closer to the truth than any single measurement, and the error shrinks as 
1
/
n
1/ 
n
​
 . This is the ancestor of the central limit theorem — a result that says, roughly, "the average of many small random errors tends to behave like a smooth bell curve, even when the individual errors don't." That fact is what justifies averaging in the first place.

So the mean was not invented as a pure abstraction. It was invented because a working astronomer needed a way to fuse noisy data, tried averaging, and it worked. Every "average" you compute today — every neural network's loss, every benchmark score, every dashboard metric — is a direct descendant of that.

Why Not the Median, the Mode, or the Most Trusted Single Reading?
Three alternatives existed before the mean became standard, and each is used somewhere today.

Method	What it does	Why the mean usually wins
Pick the best single reading	Use the measurement from the most trusted source	Ignores all the other data; subjective
Mode (most common value)	Pick the value that appears most often	Useful for discrete categories; meaningless for continuous data with no exact repeats
Median (middle value)	Pick the middle value when sorted	Robust (= not easily fooled by extreme values) but not differentiable (= you can't take its derivative cleanly), so it doesn't play well with calculus-based optimization
A few terms in that table will recur in this course; let's pin them now.

An outlier is a single data point that lies far from the rest — a millionaire in a list of regular salaries, a corrupted image in a dataset of clean ones. Sometimes outliers are real and important; sometimes they're errors that shouldn't dominate.
Skewed data is data whose distribution is lopsided — most values bunched on one side, a long tail on the other. Income is skewed (most people earn modestly, a few earn a lot). Heights are not skewed (roughly symmetric around the average).
Robust means "still gives a sensible answer when a few data points are weird." A robust statistic is not yanked around by outliers.
Differentiable means "has a smooth slope at every point" — you can compute the derivative without hitting a sharp corner. Calculus and gradient descent both require differentiability, which is why we obsess over it.
The differentiability point is the modern reason the mean dominates ML. Every gradient-based learning algorithm — every transformer, every Llama, every Claude — needs to take derivatives of its loss. The mean is smooth; the median is not. Even when the median is statistically better (skewed data, outliers), engineers usually pick the mean because it lets the optimizer move.

This trade-off — robustness vs differentiability — recurs everywhere in this course. Holding both objections in your head when you pick a loss function is most of the wisdom.

The Balance Point Interpretation
The mean has a beautiful physical interpretation. Imagine each data point is a unit weight placed on a number line. The mean is the point where the line balances.

Six data points (65, 72, 80, 85, 88, 90) on a number line with a triangular fulcrum at the mean of 80
Six data points (65, 72, 80, 85, 88, 90) on a number line with a triangular fulcrum at the mean of 80

Mathematically: the deviations from the mean always sum to zero. (A deviation is just the signed distance from the mean — how far above or below the average each data point is. Positive deviations are above; negative are below.)

∑
i
=
1
n
(
x
i
−
x
ˉ
)
=
0
∑ 
i=1
n
​
 (x 
i
​
 − 
x
ˉ
 )=0

This is a definition of the mean, not a coincidence. For our test scores:

(
72
−
80
)
+
(
85
−
80
)
+
(
90
−
80
)
+
(
65
−
80
)
+
(
88
−
80
)
(72−80)+(85−80)+(90−80)+(65−80)+(88−80) 
=
−
8
+
5
+
10
−
15
+
8
=
0
=−8+5+10−15+8=0

The negative deviations cancel the positive ones perfectly. The mean is, by construction, the balance point.

This property is why the mean shows up everywhere in physics and engineering — center of mass, center of charge, weighted average of forces. It's also why "subtract the mean to center your data" is the first preprocessing step in nearly every ML pipeline.

When the Mean Lies
The mean is famously deceptive when the data is skewed (lopsided, with a long tail in one direction) or has outliers (extreme single values that lie far from the rest).

Example. Salaries at a small startup: 
{
40
,
45
,
50
,
50
,
55
,
60
,
1000
}
{40,45,50,50,55,60,1000} (in thousands of dollars).

x
ˉ
=
40
+
45
+
50
+
50
+
55
+
60
+
1000
7
=
1300
7
≈
186
x
ˉ
 = 
7
40+45+50+50+55+60+1000
​
 = 
7
1300
​
 ≈186

The mean salary is $186k. But six out of seven employees earn under $60k. The CEO's million-dollar salary single-handedly dragged the mean above what 86% of employees actually make.

This is the most famous trap in elementary statistics: the mean is sensitive to outliers. A single extreme value can pull the average arbitrarily far from "where most of the data is".

In machine learning, the same trap appears. If your training data has a few mislabeled examples with huge errors, mean squared error will be dominated by those few outliers and the model will spend its capacity correcting them at the expense of the bulk of the data. We'll come back to this in Lesson 3 when we discuss MSE's failure modes.

The Median
The median is the middle value when the data is sorted.

For our salaries, sorted: 
{
40
,
45
,
50
,
50
,
55
,
60
,
1000
}
{40,45,50,50,55,60,1000}. The middle (4th of 7) is 50.

The median salary is $50k — much closer to what most employees actually make. The CEO's outlier no longer dominates because the median doesn't care about magnitudes, only ranks. Move $1000 to $10000 to $10 million; the median stays $50.

For an even number of points, the median is the average of the two middle values.

{
1
,
3
,
5
,
9
}
  
⟹
  
median
=
3
+
5
2
=
4
{1,3,5,9}⟹median= 
2
3+5
​
 =4

Where the Median Came From
The median, oddly enough, was invented after the mean — and for a specific reason.

In the early 1800s, Adolphe Quetelet, a Belgian astronomer who turned to social science, started collecting data on human populations: heights, chest sizes, incomes, criminal-conviction rates. He averaged everything in sight. For some quantities — height, chest size — the average worked perfectly. The data clustered around the mean and tapered off symmetrically; the average described "the typical Belgian" well.

For other quantities — wealth, income — Quetelet's averages were nonsense. The mean Belgian wealth was much higher than what any actual median Belgian had. A handful of wealthy aristocrats dragged the average up. Reporting "the average wealth" was misleading by orders of magnitude.

In 1881, Francis Galton (Darwin's cousin and an obsessive measurer of everything) coined the word median for the middle value of a sorted list, precisely to address this problem. The median was the answer to "what does a typical person look like?" when the mean was being yanked around by extremes.

So the median was invented as a response to the mean's failure on skewed data. This is the pattern of mathematics: the existing tool fails on a real problem, someone names a new tool that handles the case, and both stay in the toolbox forever.

Today, when journalists report "median household income" instead of "mean household income", they are using Galton's invention. So is anyone reporting median latency on a service (the average gets dominated by a few slow requests). So is robust statistics in general.

Mean vs Median: When to Use Which
Situation	Use the mean	Use the median
Data is roughly symmetric, no outliers	✓	(either works)
Data is skewed (income, wealth, file sizes)		✓
Outliers are real and you want them to matter	✓	
Outliers are noise / mislabels and shouldn't dominate		✓
You need to do further math (calculus, gradients)	✓	
That last row is why the mean dominates ML. The mean is differentiable — you can take its derivative with respect to anything. The median has a derivative of zero almost everywhere and a discontinuity (a sudden jump) at the middle, which makes gradient-based optimization (the family of methods, like gradient descent, that nudge parameters in the direction that decreases a loss) unhappy. When neural networks need a "summary number" for a loss (the single number the model is trying to make smaller during training), they nearly always pick the mean.

The Mean as the Best Constant Predictor
Here is a fact that will reappear, in dressed-up form, throughout this course.

If you must predict every value in a dataset using a single constant, and you measure error by squared deviation, the constant that minimizes total error is the mean.

That is, among all constants 
c
c, the one that minimizes

total squared error
(
c
)
=
∑
i
=
1
n
(
x
i
−
c
)
2
total squared error(c)=∑ 
i=1
n
​
 (x 
i
​
 −c) 
2
 

is 
c
=
x
ˉ
c= 
x
ˉ
 .

This is not a definition of the mean. It is a theorem. The mean is the optimal constant predictor under squared error. We'll prove this in Lesson 3 — it's a one-line calculus exercise — and the proof gives you the entire intuition behind why mean squared error is the canonical regression loss.

(For absolute error, the optimal constant is the median. Different loss → different optimal predictor. This connection between loss functions and the statistics they "produce" is one of the deepest unifying ideas in machine learning.)

A Programmer's Definition
In code, both are one-liners:

python
Copy
import numpy as np

scores = np.array([72, 85, 90, 65, 88])
print(scores.mean())            # 80.0
print(np.median(scores))         # 85.0
Note for the test scores, the median is 85, not 80. Why? Because the scores aren't symmetric around 80 — there's one notably low score (65) pulling the mean down. The median is robust to that.

Worked Example — A Model's Predictions
Suppose a regression model predicts house prices. The true prices are 
{
200
,
250
,
280
,
310
,
320
,
350
}
{200,250,280,310,320,350} (thousands). The model predicts 
{
210
,
240
,
290
,
300
,
330
,
360
}
{210,240,290,300,330,360}.

The error for each house: 
{
10
,
−
10
,
10
,
−
10
,
10
,
10
}
{10,−10,10,−10,10,10} (predicted minus true).

Mean error: 
10
−
10
+
10
−
10
+
10
+
10
6
=
20
6
≈
3.3
6
10−10+10−10+10+10
​
 = 
6
20
​
 ≈3.3.

The mean error is small ($3.3k) — but this is misleading. The model does make mistakes; they just happen to cancel out in sign. To measure "how much the model is off", we cannot use the raw mean — positive and negative errors fool it.

This is why we square the errors before averaging. Squaring eliminates sign cancellation. That's the leap from "mean" to "variance" to "MSE", and it is the subject of the next two lessons.

Try It Yourself
1. Compute the mean and median of 
{
1
,
2
,
2
,
2
,
3
,
4
,
100
}
{1,2,2,2,3,4,100}. Which is more representative of "typical" values? Justify in one sentence.

2. A factory produces parts with measurements 
{
10.1
,
10.2
,
9.9
,
10.0
,
10.3
}
{10.1,10.2,9.9,10.0,10.3}. The spec says parts should average exactly 10.0. Compute the mean. Is the factory on spec?

3. Show by direct calculation that for 
{
2
,
4
,
6
,
8
}
{2,4,6,8}, the deviations from the mean sum to zero.

4. A model's error vector across 5 examples is 
{
−
3
,
2
,
−
1
,
4
,
−
2
}
{−3,2,−1,4,−2}. Compute the mean error. What does this single number tell you, and what does it not tell you about the model's quality?

What This Buys You
Mean = sum divided by count. The balance point of the data. Sensitive to outliers, but differentiable, which makes it the default summary in ML.
Median = middle value when sorted. Robust to outliers, but not differentiable, so used less often in loss functions.
The mean is the optimal constant predictor under squared error — a theorem we'll prove in Lesson 3.
Raw mean of errors can lie: positive and negative errors cancel. To measure "how off" we need to square first. That's the next lesson.