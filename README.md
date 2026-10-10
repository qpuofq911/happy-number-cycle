# happy number: what the 4 → 16 → 37 loop actually means, and how to solve LeetCode 202 without memorizing a solution

Search for "happy number" and you land in two different worlds. One is recreational math: a list of curious integers, a Wikipedia page, a Numberphile video. The other is a LeetCode Easy problem that shows up in phone screens more often than its difficulty suggests.

Most people typing the phrase want one of two things: the definition plus a few examples, or working code for LeetCode 202. This covers both, including the part most write-ups skip — why *every* unhappy number in the universe funnels into the same eight-number loop.

## What makes a number happy

Take any positive integer. Replace it with the sum of the squares of its digits. Repeat. If you eventually hit 1, the number is happy. If you fall into a cycle that never contains 1, it isn't.

19 is the standard example:


1² + 9² = 82
8² + 2² = 68
6² + 8² = 100
1² + 0² + 0² = 1   → happy


7 works too: 7 → 49 → 97 → 130 → 10 → 1. So does 100, which gets there in a single step.

The happy numbers below 100 are:

**1, 7, 10, 13, 19, 23, 28, 31, 32, 44, 49, 68, 70, 79, 82, 86, 91, 94, 97**

That's 19 of them, and 100 makes it an even 20 in the first hundred integers. The gaps are erratic — clusters of two or three, then long dry stretches. There's no formula that generates the sequence, which is exactly why the coding problem exists.

## Why every unhappy number ends up in the same loop

Here's the part that makes this problem more interesting than it looks.

Start with 2 and keep going:


2 → 4 → 16 → 37 → 58 → 89 → 145 → 42 → 20 → 4 ...


Once you reach 4, you're trapped. The sequence cycles through **4, 16, 37, 58, 89, 145, 42, 20** and returns to 4 forever. The same eight numbers show up if you start from 3, 5, 85, 999, or any other unhappy number. There is exactly one non-trivial loop in this whole process.

Why? Because the sum of squared digits shrinks numbers fast. For an integer with *d* digits, the largest possible result is 81*d*. LeetCode 202 accepts inputs up to 2³¹ − 1, which is ten digits, so the very first step lands at 810 or below. Any three-digit number maxes out at 9² + 9² + 9² = 243. So after at most two steps you're inside the range [1, 243] and you never leave it.

A finite range with a deterministic rule means the sequence must either reach 1 or repeat. It happens that 1 and the 4-cycle are the only two attractors available. This is not a coincidence you need to prove in an interview, but knowing it explains why the two standard solutions work at all — the search space collapses to a couple hundred possible states.

## Two ways to solve LeetCode 202

The problem statement is short: given a positive integer `n`, return `true` if it's happy, `false` otherwise. Both accepted approaches come down to cycle detection.

### Approach 1: track visited numbers with a hash set

The straightforward version stores every number you've seen. Repeat a number, and you're in the loop.

python
def is_happy(n: int) -> bool:
    seen = set()
    while n != 1 and n not in seen:
        seen.add(n)
        total = 0
        while n > 0:
            n, digit = divmod(n, 10)
            total += digit * digit
        n = total
    return n == 1


The inner loop is the digit manipulation people fumble: `divmod(n, 10)` pulls off the last digit and shaves it off `n` in one call. Once `n` hits 0, `total` is the next number in the sequence.

The Java equivalent, using the return value of `Set.add` to avoid a second lookup:

java
class Solution {
    public boolean isHappy(int n) {
        Set<Integer> seen = new HashSet<>();
        while (n != 1 && seen.add(n)) {
            n = next(n);
        }
        return n == 1;
    }

    private int next(int n) {
        int sum = 0;
        while (n > 0) {
            int d = n % 10;
            sum += d * d;
            n /= 10;
        }
        return sum;
    }
}


`seen.add(n)` returns `false` if the value was already there, so the loop exits on a repeat without an explicit `contains` check. Small thing, but interviewers notice it.

### Approach 2: Floyd's cycle detection

If the interviewer asks for constant extra space, the hash set is out. Floyd's tortoise-and-hare works instead: one pointer advances one step at a time, the other advances two. If they meet somewhere other than 1, there's a cycle — which in this problem means the number isn't happy.

python
def sum_of_squares(n: int) -> int:
    total = 0
    while n > 0:
        n, digit = divmod(n, 10)
        total += digit * digit
    return total

def is_happy(n: int) -> bool:
    slow = n
    fast = sum_of_squares(n)
    while fast != 1 and slow != fast:
        slow = sum_of_squares(slow)
        fast = sum_of_squares(sum_of_squares(fast))
    return fast == 1


The exit condition needs care. The loop stops when either the fast pointer reaches 1 (happy) or the two pointers collide (cycle, unhappy). Returning `fast == 1` covers both cases correctly. A common bug is returning `slow == fast` or checking the pointers after the wrong update, which flips the answer for numbers like 19.

There's also a shortcut some solutions use: since every unhappy number passes through 4, you can loop `while n != 1 and n != 4` and skip the pointers entirely. It's valid — the eight-number cycle is fixed and well documented. It's also the kind of answer that invites the follow-up "why is 4 special?" so if you use it, be ready to justify it.

## Complexity, edge cases, and the mistakes that cost you the round

| Approach | Time | Space | When to use it |
| --- | --- | --- | --- |
| Hash set | O(log n) per step, bounded overall | O(log n) — the visited set | Default answer. Easiest to explain. |
| Floyd's two pointers | Same | O(1) | When space is constrained or the pattern is the point |

The time complexity looks scarier than it is. Each digit-sum step costs O(log n) for a number with log n digits, but the sequence collapses into [1, 243] almost immediately, so the total number of iterations is bounded by a small constant. That's why the loop never runs long, regardless of how big the input is.

Things that actually trip people up:

- **`n = 1`** must return `true` immediately. The sequence is already at its fixed point.
- **Zero is not input.** The constraint starts at 1. For what it's worth, 0 would never reach 1 — its digit sum is always 0.
- **Don't reset the accumulator inside the wrong loop.** The digit-sum helper needs a fresh `total = 0` on every call, and the outer loop must not add to it.
- **Don't confuse this with the digit-sum problem** (LeetCode 258). That one adds digits; this one squares them.

## The follow-up questions that actually get asked

Once you've written a working solution, the real interview starts. Reasonable next questions:

**"What if I gave you a stream of numbers and asked you to return all the happy ones under a million?"** Precomputation. Compute the status for every value up to 243 first, then for any input walk the sequence until you land inside that precomputed table. This is a standard dynamic programming twist and it's why people call 243 the magic bound.

**"How is this different from detecting a cycle in a linked list?"** It isn't, structurally. Same pattern, different node representation. If you've internalized fast-and-slow pointers for Linked List Cycle, the transfer is direct — which is the whole argument for learning patterns instead of individual solutions.

**"Can you prove there's only one loop?"** You can sketch it: the range is finite after two steps, so enumerate [1, 243] and observe that everything either reaches 1 or joins the 4-cycle. A brute-force script settles it in milliseconds.

If you want to see this worked through step by step with the state of the visited set printed after every iteration, 👉 [the AlgoMonster editorial for problem 202](https://algo.monster/liteproblems/202?fpr=li69) walks the trace for `n = 2` line by line, showing the set growing to `{2, 4, 16, 37, 58, 89, 145, 42, 20}` before the repeat on 4 fires and the function returns `false`. AlgoMonster files this problem under Two Pointers (Advanced), which tells you something about their intent — they treat the Floyd's variant as the version worth knowing, not the hash set.

## Where a pattern-based platform fits in

Solving 202 is not hard. Recognizing *why* it belongs next to Linked List Cycle and Find the Duplicate Number is the skill that transfers, and it's the thing people miss when they grind problems one at a time.

AlgoMonster is built around that framing. It's a text-and-diagram platform from ex-Google engineers and competitive programmers — no video, roughly 1,000+ illustrations, 500+ lessons across six courses, 3,000+ editorials, covering about 48 coding patterns. The pitch is that a few dozen reusable templates cover most interview questions, so you spend your time learning decision logic rather than accumulating solved problems.

Two features matter for a problem like this one. The AI assistant is wired through the content rather than bolted on as a separate chatbot: you submit code and it points at the failing case and explains why, which is more useful than a 👍/👎 on a submission verdict. Speedrun gives you timed runs over the patterns you've already covered, which is the closest the platform gets to interview pressure.

Worth being clear about the limits. There are no video walkthroughs, the problem count sits in the low hundreds rather than thousands, and system design and behavioral coverage are introductory at best. If your prep plan includes an L5 system design round, that lives somewhere else. If your bottleneck is "I know the syntax but I can't tell which pattern a new question wants," that's the gap it's aimed at — and the platform claims the curriculum can be worked through in 4–8 weeks at an hour or two a day.

## AlgoMonster plans and what they cost

AlgoMonster has no monthly option. There's free sample content and two paid tiers.

| Plan | What you get | Price | Billing | Get it |
| --- | --- | --- | --- | --- |
| Free | Limited access to a subset of lessons and problems | $0 | — | [Explore the free content](https://algo.monster?fpr=li69) |
| Pro – Yearly | All patterns, lessons and problems; unlimited AI Assistant; unlimited Speedrun; company questions updated monthly; 500+ lessons across 6 courses, 1,000+ illustrations, 3,000+ editorials; member discount on live bootcamps; bonus Foundation and Build-an-LLM courses | $5.84/mo — $69.99 billed annually | 12 months, cancel or upgrade anytime | [Get Pro Yearly](https://algo.monster/subscribe?fpr=li69) |
| Pro – Lifetime | Everything in the yearly plan plus lifetime access and all future content updates | $89 | One-time | [Get Pro Lifetime](https://algo.monster/subscribe?fpr=li69) |

A note on the numbers: those are the rates the subscribe page showed when this was written. AlgoMonster runs sales often enough that coupon sites disagree wildly with each other about the "real" price, so treat any figure you see on a third-party page as suspect and check the checkout page yourself.

The annual plan makes sense if you have an interview on the calendar and intend to stop there. Lifetime at $89 is cheaper than a year of most competitors' subscriptions, and if you expect to change jobs again in a couple of years, the math is not close.

## Quick answers

**Is 1 a happy number?** Yes. 1² = 1, so it sits at the fixed point immediately.

**Is 4 a happy number?** No — it's the entry point to the unhappy cycle, which is why some solutions use it as a sentinel.

**Is 202 happy?** No. 202 → 8 → 64 → 52 → 29 → 85 → 89 → 145 → ... straight into the loop. Slightly ironic given the problem number.

**Do I need to memorize the 4 → 16 → 37 → 58 → 89 → 145 → 42 → 20 cycle?** Not for the hash set version. It helps for the constant-space version, or if you want to explain the precomputation trick convincingly.

**Which language shows up most in interviews?** Python and Java, in that order, for this problem. The digit-extraction loop is where most of the mistakes happen, so practice `divmod(n, 10)` or `n % 10` / `n //= 10` until it's automatic.

Two steps in, every input to this problem is a number below 244. That's the whole trick — the problem looks open-ended and isn't. Solve it once with a set, once with two pointers, and you've learned a pattern that reappears in half a dozen other questions.
