# Chapter 4 - Comments
Comments are not “pure good.” Indeed, they are, at best, a necessary evil.
## Comments Do Not Make Up for Bad Code
Clear and expressive code with few comments is far superior to cluttered and complex code with lots of comments.
## Explain Yourself in Code
Don't Use a Comment when you can use a Function or a Variable
## Good Comments
- Legal Comments
- Informative Comments
- Explanation of Intent
- Clarification
- Warning of Consequences
- TODO Comments
- Amplification (to emphasize the importance of something that may otherwise seem inconsequential)
- Javadocs in Public APIs
## Bad Comments
- Mumbling (we should clearly explain ourselves in comments instead of confusing the readers)
- Redundant Comments (if comment is longer than the code it is explaining, it probably shouldn't be there)
- Misleading Comments
- Mandated Comments (comments shouldn't be required for every function and variable)
- Journal Comments (don't make our code our diary)
- Noise Comments
- Position Markers (`///// Actions /////`)
- Closing Brace Comments
- Attributions and Bylines (VCS can track who adds the code, there is no need to put our names at every function we write)
- Commented-Out Code (we can use VCS to keep track of old code)
- HTML Comments (use a tool instead because HTML is hard to read)
- Nonlocal Information (don’t offer system-wide information in the context of a local comment)
- Too Much Information
- Inobvious Connection (the connection between a comment and the code it describes should be obvious.)
- Function Headers (short functions don’t need much description. A well-chosen name for a small function that does one thing is usually better than a comment header.)
- Javadocs in Nonpublic Code