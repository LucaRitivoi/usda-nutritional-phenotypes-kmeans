# What I Actually Did (No Math Degree Required)

*A plain-language companion to [TECHNICAL_REPORT.md](TECHNICAL_REPORT.md) and [README.md](README.md).*

---

Food packaging is confusing. One product says "low fat," another says "high protein," another says "diet," and none of it actually tells you the full picture or lets you fairly compare one food to another. This project is a way to cut through that: instead of trusting whatever claim happens to be printed on the package, you can look at what a food's nutrients actually are and see where it really lands, no confusion, no spin, simply the numbers. It offers a more standard, trustworthy way to actually understand what you're buying.

This project groups foods together based only on their nutritional composition, with no bias from marketing and no attention paid to anyone's opinions about nutrition. It does this with the help of a computer, which sorts everything objectively through math alone.

## How does a computer sort food without knowing anything about food?

Instead of picking team captains, imagine a few random people are simply told to stand somewhere as a starting point. Everyone else walks over and stands next to whichever starting person they feel most similar to. Once everyone's grouped up, each group moves to stand in the actual middle of their group, not around the original starting person anymore, but around wherever the "average" person in that group would be. Now that the middle has moved, a few people near the edges might realize they're actually standing closer to a different group's new middle, so they walk over and join that one instead. Everyone keeps re-checking and shifting like this, a few rounds of reassessing who they're standing closest to, until nobody moves anymore and the groups settle.

That's what happens to 7,058 foods here, simply with numbers (Calories, Protein, Fat, Carbs, Sugar, Sodium) standing in for "who you feel closest to." The computer doesn't know or care what any of these foods are called, how they're marketed, or what food group they're "supposed" to belong to. It simply looks at the numbers and lets the foods sort themselves.

One more thing worth knowing: the computer doesn't decide how many groups to make on its own. It gets tested at every number from 1 to 10, and I looked at those results myself to decide that 4 groups struck the right balance, enough to capture real differences, but not so many that the groups become too specific to explain clearly.

## What it actually found

The computer found four groups. One is high in protein (chicken, fish, lean meats). One is dense in fat and calories (oils, butters, shortenings). One is high in sodium and carbohydrates, basically what most people would call junk food. And one is low across nearly every measurement, mostly vegetables and fruit.

The groupings hold up in ways that would be hard to guess from category alone. Dried walrus meat and a plant-based sausage alternative end up in the same group, because their protein content puts them there, regardless of one being a traditional meat and the other being marketed as a meat substitute. Candy and most cereal end up in the same group too, because both are loaded with sugar, even though cereal is usually marketed and perceived as the healthier option of the two.

## Why this actually matters

We've all struggled with the uncertainty of what we're buying, especially when we look at an ingredient list and feel like it's in a foreign language. This project gets rid of that; it brings all of the foods to one collective language that makes it easier for customers to know what they are buying based off of real, unbiased data.
