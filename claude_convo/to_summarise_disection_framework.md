Short answer: the data doesn't support "architecture instead of fundamentals." It supports "architecture becomes more valuable, and fundamentals become the thing that lets you use AI safely." No study directly compares the career value of architecture knowledge against DSA, so the evidence below is indirect, but it points in a fairly consistent direction.

**Evidence that routine coding is being automated, and judgment matters more**

The strongest labor-market data is Stanford's "Canaries in the Coal Mine?" paper, based on ADP payroll records. Employment for software developers aged 22 to 25 had declined nearly 20% by July 2025, while opportunities for more experienced developers in the same companies grew or remained stable. One common reading is that AI replaces codified, textbook-style knowledge first, while experience-based judgment (which includes design and architecture sense) holds its value. That interpretation is plausible but not proven; the paper shows a correlation.

**Evidence that system foundations determine whether AI helps**

Google's DORA 2025 report, which surveyed nearly 5,000 professionals, found that AI doesn't fix a team; it amplifies what's already there, and the greatest return comes not from the AI tools themselves but from the quality of internal platforms, the clarity of workflows, and the alignment of teams. It also found that AI adoption continues to have a negative relationship with software delivery stability. In other words, AI speeds up code production, so well-structured systems become more important, not less. That's a point in favor of architecture knowledge.

**Evidence that fundamentals still matter, maybe more**

Anthropic's randomized trial on learning a new Python library found the AI group averaged 50% on the quiz, compared to 67% in the hand-coding group, and the largest gap appeared in debugging, suggesting AI assistance particularly undermines the ability to identify when code is wrong and understand why. That's exactly the skill you need when AI writes most of your code. Developers seem to sense this: in the Stack Overflow 2025 survey, only 3% of developers report "high trust" in AI-generated output. Someone has to verify that output, and verifying requires understanding complexity, data structures, and the language.

Productivity gains also aren't automatic. METR's trial found that experienced developers took 19% longer to complete issues when allowed to use AI, and even after experiencing the slowdown, they still believed AI had sped them up by 20%. One caveat: METR now labels the result as historical, saying it no longer necessarily reflects current AI tools or current developer workflows.

**What this means for you**

The two aren't really competing. Architecture is where the value is shifting: AI can generate a class or function well, but deciding how a system is decomposed, where data lives, and which trade-offs to make depends on context AI often doesn't have. But you can't make good architectural decisions without fundamentals, since trade-offs about performance, memory, and consistency are reasoned about in terms of complexity and data structures.

What's probably losing value is DSA as memorized trivia, such as recalling tricky algorithm implementations for interviews. DSA as understanding, knowing why a hash map beats a list for lookups or spotting an accidental O(n²) loop in generated code, remains essential.

One practical takeaway from the Anthropic study applies directly to how you're learning right now: developers who used AI for conceptual questions scored 65% or higher, while those delegating code generation to AI scored below 40%. Using AI to explain concepts, like you've been doing in this conversation, supports learning. Having it write code you don't understand undermines it.

Keep in mind that these are early studies, several with small samples and older tools, so treat them as signals rather than settled conclusions.