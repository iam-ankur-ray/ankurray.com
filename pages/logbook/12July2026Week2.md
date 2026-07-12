# Coding promblems.

## Problem - Need to find out the ascii code of a character of smaller alphabet(97) and starting it index from 1 to add those index along with that character out of character .i.e, abc = a1b2c3
- Core Idea - Get the ascii character of alphabet - character itself + 1 to initiate the index and then adding the character by appending null string.
- Algorithm Discovery - We can use StringBuilder to avoid creating copy of immutable strings in the loop to fill the empty string.
- Key Pattern - Within loop create int value = char - char + 1 and append in the Stringbuilder for charAt(i)
- Lessons Learned- We can add character in String and better to use StringBuilder if it is changing.

## Problem - We need to remove am or pm out of string created with three characters apm multiple times and check if after multiple iterations of removing am or pm from any index if the string is empty or not.
- Core Idea - m = a + n;
- Algorithm Discovery - We don't need any iteration as odd value can be discarded and for remaining characters above formula will work.