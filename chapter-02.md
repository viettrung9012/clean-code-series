# Chapter 2 - Meaningful Names
As programmers, we give names to everything - variables, functions, classes, etc. Because we do so much of it, we'd better do it well.

## Some (recommended) rules for creating good names
- Use Intention-Revealing Names - `int d` vs `int pollingInterval`
- Avoid Disinformation 
  - don't use `accountList` if it might not be a list, `accounts` is better
  - I and l looks the same, imagine I18n and l10n in the same program. Be consistent with casing
- Make Meaningful Distinctions - It is not sufficient to add number series or noise words e.g. `name1` and `name2` or `theUser`
- Use Pronouceable Names
- Use Searchable Names - avoid single letter name and magic number
- Avoid Encodings
  - Hungrarian Notation (name + type e.g. bookObject, authorString)
  - Member prefixes (m_private)
  - Interfaces and Implementation (IBook vs BookImpl)
- Avoid Mental Mapping - don't make reader spend time deducing names
- Class Names
  - should be noun or noun phrases
- Method Names
  - should be verb or verb phrases
  - accessors, mutators, and predicates should be prefixed with `get`, `set` and `is`
  - use factory/builder method when constructor is overloaded (IntelliJ displays the properties' names but Git doesn't)
- Don’t Be Cute (`delete()` instead of `goodBye()`)
- Pick One Word per Concept - don't mix e.g. `fetch`, `retrieve` and `get` usually mean the same action
- Don’t Pun - is `add` an arithmetic operation `sum` or a list operation `append`
- Use Solution Domain Names - add patterns, data structures, and terms that other programmers are familiar with e.g. Message**Builder**, Message**Queue**
- Use Problem Domain Names - when solution domain is not possible, use names those relate to the problem domain so that the next programmer can ask a domain expert what it means
- Add Meaningful Context - if there are more than one possible meaning for the chosen name in the scope, it is better to add context to name
- Don’t Add Gratuitous Context - don't prefix for all classes in the same module with the module name or abbreviation e.g. BookAuthor, BookTitle, BookDescription, etc. It is better to organized those into Book package/directory