# Question 1: Why is HashMap lookup considered O(1) on average?

HashMap internally uses an array of buckets. When you call get(key), it runs the key through a hash function, which produces an integer. That integer is then mapped to a bucket index in the array using index = hash % array_size. Accessing an array by index is O(1) — no iteration needed.
It's called average O(1) because in the case of hash collisions (two keys landing in the same bucket), it has to search a short linked list or tree within that bucket — making worst case O(n) or O(log n).  

The Library Analogy
Imagine a library with 1000 numbered shelves (0 to 999).
When you want to store a book titled "Java", the librarian runs the title through a formula:

"Java" → formula → shelf number 42

Now the book sits on shelf 42.
Later, when you ask "where is the Java book?" — the librarian doesn't walk through all 1000 shelves. He just runs the same formula:

"Java" → formula → shelf 42 → book found instantly.

That formula is the hash function. That shelf number is the bucket index. Going directly to a shelf number = O(1).

Why "Average" O(1)?
Sometimes two books get assigned the same shelf. Now shelf 42 has 2 books and you have to look through both. That's a collision. If somehow all 1000 books land on shelf 42, you'd have to scan all 1000 — that's worst case O(n).
But in practice, a good hash function spreads books evenly, so collisions are rare — hence average O(1).


