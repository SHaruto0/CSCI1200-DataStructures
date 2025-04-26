# Lecture 15 --- Associative Containers (Maps), Part 1

Associative Containers (STL Maps)
- STL Pairs
- STL Maps: associative containers for fast insert, access and remove
- Example: Counting word occurrences
- Map iterators
- Map member functions: operator[], find, insert, erase.
- Efficiency
- STL maps vs. STL vectors vs. STL lists

## 15.1 STL Pairs

- std::pair is a templated struct with just two members, called first and second. 

```cpp
template <typename T1, typename T2>
struct pair {
    T1 first;
    T2 second;

    // Constructors
    constexpr pair();
    constexpr pair(const T1& x, const T2& y);
    
    // Other member functions...
};
```

**struct** in C++ is identical to class except that members are public by default, std::pair is implemented as a struct to allow direct access to its first and second members without requiring explicit getter functions.  
- To work with pairs, you must #include &lt;utility&gt;. Note that the header file for maps (#include &lt;map&gt;) itself includes utility, so you don't have to include utility explicitly when you use pairs with maps.
- Here are simple examples of manipulating pairs:
```cpp
std::pair<int, double> p1(5, 7.5);
std::pair<int, double> p2 = std::make_pair(8, 9.5);
p1.first = p2.first;
p2.second = 13.3;
std::cout << p1.first << " " << p1.second << std::endl;
std::cout << p2.first << " " << p2.second << std::endl;
p1 = p2;
std::pair<const std::string, double> p3 = std::make_pair(std::string("hello"), 3.5);
p3.second = -1.5;
// p3.first = std::string("illegal"); // (a)
// p1 = p3; // (b)
```
- The function std::make_pair creates a pair object from the given values. It is really just a simplified
constructor, and as the example shows there are other ways of constructing pairs.
- Most of the statements in the above code show accessing and changing values in pairs.
The two statements at the end are commented out because they cause syntax errors:
- In (a), the first entry of p3 is const, which means it can’t be changed.
- In (b), the two pairs are different types! Make sure you understand this.

## 15.2 STL Maps: Associative Containers

- STL maps store pairs of “associated” values.
- e.g., An association between a string, representing a word, and an int representing the number of times that
word has been seen in an input file.
- e.g., An association between a string, representing a word, and a vector that stores the line numbers from a
text file on which that string occurs.
- e.g., An association between a phone number and the name of the person with that number.
- e.g., An association between a class object representing a student name and the student’s info.
 A particular instance of a map is defined (declared) with the syntax:

```cpp
std::map<key_type, value_type> var_name
```
- In our first two examples above, key type is a string. In the first example, the value type is an int and in
the second it is a std::vector&lt;int&gt;.
- Entries in maps are pairs: std::pair&lt;const key_type, value_type&gt;, the const is needed to ensure that the keys aren’t changed! This is crucial because maps are sorted by keys!
- Map iterators refer to pairs.
- Map search, insert and erase are all very fast: O(log n) time, where n is the number of pairs stored in the map.
<!-- Note: The STL map type has similarities to the Python dictionary, Java HashMap, or a Perl hash, but the
data structures are not the same. The organization, implementation, and performance is different. In a couple
weeks we’ll see an STL data structure that is even more similar to the Python dictionary.-->

## 15.3 Counting Word Occurrences

First, let’s see how some of this works with a program to count the occurrences of each word in a file. Here’s a simple and elegant solution to this problem using a map:

```cpp
#include <iostream>
#include <map>
#include <string>
int main() {
	std::string s;
	std::map<std::string, int> counters; // store each word and an associated counter
	// read the input, keeping track of each word and how often we see it
	while (std::cin >> s){
		++counters[s];
	}
	// write the words and associated counts
	std::map<std::string, int>::const_iterator it;
	for (it = counters.begin(); it != counters.end(); ++it) {
		std::cout << it->first << "\t" << it->second << std::endl;
	}
	return 0;
}
```

## 15.3 Maps: Uniqueness and Ordering

- Maps are ordered by increasing value of the key. Therefore, there
must be an operator< defined for the key.
- Once a key and its value are entered in the map, the key can’t be
changed. It can only be erased (together with the associated value).
- Duplicate keys can not be in the map.

## 15.5 Maps: operator[]

- We’ve used the [] operator on vectors, which is conceptually very simple because vectors are just resizable
arrays. Arrays and vectors are efficient random access data structures.
- But operator[] is actually a function call, so it can do things that aren’t so simple too, for example:

```cpp
++counters[s];
```

- For maps, the [] operator searches the map for the pair containing the key (string) s.  
  
  – If such a pair containing the key is not there, the operator:
    1. creates a pair containing the key and a default initialized value,
    2. inserts the pair into the map in the appropriate position, and
    3. returns a reference to the value stored in this new pair (the second component of the pair).  
    This second component may then be changed using operator++.  
  
  – If a pair containing the key is there, the operator simply returns a reference to the value in that pair.
- In this particular example, the result in either case is that the ++ operator increments the value associated with
string s (to 1 if the string wasn’t already it a pair in the map).
- For the user of the map, operator[] makes the map feel like a vector, except that indexing is based on a
string (or any other key) instead of an int.
- Note that the result of using [] is that the key is ALWAYS in the map afterwards.

## 15.6 Map Iterators

- Iterators may be used to access the map contents sequentially. Maps provide begin() and end() functions for
accessing the bounding iterators. Map iterators have ++ and -- operators.
- Each iterator refers to a pair stored in the map. Thus, given map iterator it, it-&gt;first is a const string
and it-&gt;second is an int. Notice the use of it-> , and remember it is just shorthand for (*it).

## 15.7 Map Find

- One of the problems with operator[] is that it always places a key / value pair in the map. Sometimes we
don’t want this and instead we just want to check if a key is there.
- The find member function of the map class does this for us. For example:
m.find(key);
where m is the map object and key is the search key. It returns a map iterator:
If the key is in one of the pairs stored in the map, find returns an iterator referring to this pair.
If the key is not in one of the pairs stored in the map, find returns m.end().

## 15.8 Map Insert

- The prototype for the map insert member function is:
m.insert(std::make_pair(key, value));
insert returns a pair, but not the pair we might expect. Instead it is pair of a map iterator and a bool:
std::pair<map<key_type, value_type>::iterator, bool>
- The insert function checks to see if the key being inserted is already in the map.

  – If so, it does not change the value, and returns a (new) pair containing an iterator referring to the existing
pair in the map and the bool value false.

  – If not, it enters the pair in the map, and returns a (new) pair containing an iterator referring to the newly
added pair in the map and the bool value true.

## 15.9 Map Erase

- Maps provide three different versions of the erase member function:
  - void erase(iterator p) — erase the pair referred to by iterator p.  
  - void erase(iterator first, iterator last) — erase all pairs from the map starting at first and going
up to, but not including, last.  
  - size_type erase(const key_type& k) — erase the pair containing key k, returning either 0 or 1, depending
on whether or not the key was in a pair in the map.

## 15.10 Leetcode Exercise

Given an integer array nums and an integer k, return true if there are two distinct indices i and j in the array such that nums[i] == nums[j] and abs(i - j) <= k.

We have the solution below.

```cpp
#include <iostream>
#include <vector>
#include <map>

bool containsNearbyDuplicate(std::vector<int>& nums, int k) {
        int size = nums.size();
        // create the map, map key is the value of the vector element, map value is the index of that element in the vector.
	std::map<int,int> map1;
        for(int i=0;i<size;i++){
            // if already exists
            if(map1.find(nums[i])!=map1.end()){
                if(i-map1[nums[i]]<=k){
                    return true;
                }
            }
            map1[nums[i]]=i;
        }
        return false;
}

int main() {
     std::vector<std::vector<int>> testCases = {
        {1, 2, 3, 1},       // Expected: true (nums[0] == nums[3], abs(0-3) <= k)
        {1, 0, 1, 1},       // Expected: true (nums[2] == nums[3], abs(2-3) <= k)
        {1, 2, 3, 4, 5},    // Expected: false (no duplicates)
        {1, 2, 3, 4, 1},    // Expected: true if k >= 4
        {99, 99},           // Expected: true if k >= 1
        {1, 2, 3, 4, 5, 6}, // Expected: false (no duplicates)
    };

    std::vector<int> kValues = {3, 1, 2, 4, 1, 2}; // Corresponding k values for test cases

    for (size_t i = 0; i < testCases.size(); i++) {
        std::cout << "Test Case " << i + 1 << ": ";
        bool result = containsNearbyDuplicate(testCases[i], kValues[i]);
        std::cout << (result ? "true" : "false") << std::endl;
    }

    return 0;
}
```

When running the above program, we get:
```console
$ ./a.out 
Test Case 1: true
Test Case 2: true
Test Case 3: false
Test Case 4: true
Test Case 5: true
Test Case 6: false
```

You can also find this problem on Leetcode.
- [Leetcode problem 219: Contains Duplicate II](https://leetcode.com/problems/contains-duplicate-ii/description/). Solution: [p219_contains_duplicate_ii.cpp](../../leetcode/p219_contains_duplicate_ii.cpp).

- Play this [animation](https://jidongxiao.github.io/CSCI1200-DataStructures/animations/maps/example1.html) to understand how this program works.

## 15.11 More Leetcode Exercises

- [Leetcode problem 1: Two Sum](https://leetcode.com/problems/two-sum/). Solution: [p1_twosum.cpp](../../leetcode/p1_twosum.cpp).
- [Leetcode problem 290: Word Pattern](https://leetcode.com/problems/word-pattern/). Solution: [p290_word_pattern.cpp](../../leetcode/p290_word_pattern.cpp).

