## Beginner-Level String Problems

1. Reverse a String – Reverse the characters in a string.

```java
public class ReverseString {
    public static String reverse(String input) {
        return new StringBuilder(input).reverse().toString();
    }
}
```

```python
def reverse_string(input_str):
    return input_str[::-1]
```
2. Check for Palindrome – Determine if a string reads the same backward as forward.

```java
public class PalindromeCheck {
    public static boolean isPalindrome(String input) {
        String reversed = new StringBuilder(input).reverse().toString();
        return input.equals(reversed);
    }
}
```
```python
def is_palindrome(input_str):
    return input_str == input_str[::-1]
```
3. Anagram Check – Verify if two strings contain the same characters in a different order.

```java
import java.util.Arrays;

public class AnagramCheck {
    public static boolean areAnagrams(String str1, String str2) {
        char[] a = str1.replaceAll("\\s", "").toLowerCase().toCharArray();
        char[] b = str2.replaceAll("\\s", "").toLowerCase().toCharArray();
        Arrays.sort(a);
        Arrays.sort(b);
        return Arrays.equals(a, b);
    }
}
```
```python
def are_anagrams(str1, str2):
    return sorted(str1.replace(" ", "").lower()) == sorted(str2.replace(" ", "").lower())
```
4. First Non-Repeating Character – Find the first character that does not repeat in a string.

```java
import java.util.LinkedHashMap;
import java.util.Map;

public class FirstNonRepeatingChar {
    public static Character firstNonRepeating(String input) {
        Map<Character, Integer> counts = new LinkedHashMap<>();
        for (char c : input.toCharArray()) {
            counts.put(c, counts.getOrDefault(c, 0) + 1);
        }
        for (Map.Entry<Character, Integer> entry : counts.entrySet()) {
            if (entry.getValue() == 1) return entry.getKey();
        }
        return null;
    }
}
```
```py
from collections import OrderedDict

def first_non_repeating_char(input_str):
    counts = OrderedDict()
    for c in input_str:
        counts[c] = counts.get(c, 0) + 1
    for c, count in counts.items():
        if count == 1:
            return c
    return None
```
5. Count Vowels and Consonants – Calculate the number of vowels and consonants in a string.
```java
public class VowelConsonantCount {
    public static int[] countVowelsAndConsonants(String input) {
        int vowels = 0, consonants = 0;
        input = input.toLowerCase();
        for (char c : input.toCharArray()) {
            if (Character.isLetter(c)) {
                if ("aeiou".indexOf(c) != -1) vowels++;
                else consonants++;
            }
        }
        return new int[]{vowels, consonants};
    }
}
```
```py
def count_vowels_and_consonants(input_str):
    vowels = 'aeiou'
    v = c = 0
    for ch in input_str.lower():
        if ch.isalpha():
            if ch in vowels:
                v += 1
            else:
                c += 1
    return v, c
```
6. Remove Duplicate Characters – Eliminate repeated characters, retaining only the first occurrence.
```java
public class RemoveDuplicates {
    public static String removeDuplicates(String input) {
        StringBuilder result = new StringBuilder();
        for (char c : input.toCharArray()) {
            if (result.indexOf(String.valueOf(c)) == -1) {
                result.append(c);
            }
        }
        return result.toString();
    }
}
```
```py
def remove_duplicates(input_str):
    result = ''
    for c in input_str:
        if c not in result:
            result += c
    return result
```
7. Replace Spaces with '%20' – Replace all spaces in a string with '%20', similar to URL encoding.
```java
public class ReplaceSpaces {
    public static String replaceSpaces(String input) {
        return input.replace(" ", "%20");
    }
}
```
```py
def replace_spaces(input_str):
    return input_str.replace(' ', '%20')
```
8. Case Conversion – Convert all characters in a string to uppercase or lowercase.
```java
public class CaseConversion {
    public static String toUpperCase(String input) {
        return input.toUpperCase();
    }

    public static String toLowerCase(String input) {
        return input.toLowerCase();
    }
}
```
```py
def to_upper_case(input_str):
    return input_str.upper()

def to_lower_case(input_str):
    return input_str.lower()
```
9. Check for Rotation – Determine if one string is a rotation of another.
```java
public class RotationCheck {
    public static boolean isRotation(String s1, String s2) {
        return s1.length() == s2.length() && (s1 + s1).contains(s2);
    }
}
```
```py
def is_rotation(s1, s2):
    return len(s1) == len(s2) and s2 in (s1 + s1)
```
10. String Comparison – Compare two strings for equality, considering case sensitivity.
```java
public class StringComparison {
    public static boolean compareStrings(String s1, String s2) {
        return s1.equals(s2);
    }
}
```
```py
def compare_strings(s1, s2):
    return s1 == s2
```