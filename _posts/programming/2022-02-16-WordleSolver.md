---
layout: single
title: "Wordle Solver / Solution Finder"
date: 2022-02-16 08:49:23 +0100
categories: programming
excerpt: ✏ Java 17 Wordle Solver / Solution Finder
---

# ✏ Java 17 Wordle Solver / Solution Finder

### [Source Code GitHub Link](https://github.com/71xn/algorithmsDataStructures/tree/main/wordleSolver)

### [Wordle Link](https://www.nytimes.com/games/wordle/index.html)

### [Historical Wordle](https://lookleft.github.io/wordle-history) - used for testing the algorithm

## Description

> This code aims to find a list of potential solutions to a Wordle problem, based on the known letters in wrong positions (yellow), the known letters in correct positions (green) and letters known not to be in the word (grey).
> This solution makes use of a word list of ~2500 5 letter words and a frequency table paired with user input to attempt to find an optimal solution.

### TODO

- ✔ Still to do is to add in functionality such that potential solutions are ranked based on their English language frequency to find more optimal solutions.
- ❌ Work on incorporating more information theory to rank potential solutions, rather than using a pure elimination method to find solutions, and relying on word frequency to rank potential solutions
- ❌ Convert the application to GUI, rather than console, and preferably something on the web (potentially converting the codebase to React.JS)

## Screenshots

![](/images/programming/wordle.png)

## Acknowledgements

The inspriration for this code comes from [3blue1brown](https://www.3blue1brown.com/) and his excellent videos on Wordle & information theory that can be found [here](https://www.3blue1brown.com/lessons/wordle).

Some of the data for this code and this solution came from [Grant's GitHub page](https://github.com/3b1b/videos/tree/master/_2022/wordle/data), namely the word list and the frequency list (adapted from wolfram alpha and further adapted to a txt file to read into Java).

## Data

- [Word List](https://github.com/71xn/algorithmsDataStructures/blob/ac803ac34da35f6a653583fb85c855e72a8937b8/wordleSolver/words.txt), in txt format for easy parsing into Java
- [Frequency List](https://github.com/71xn/algorithmsDataStructures/blob/ac803ac34da35f6a653583fb85c855e72a8937b8/wordleSolver/frequency.txt), in txt form to parse into a HashMap for sorting in Java.

## Code

```java
package tech.finnlestrange;

import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
import java.util.*;

public class Solution {

    List<String> words;
    Map<String, Float> frequencies;

    private void readWords(String filename) {

        words = new LinkedList<>();

        try (BufferedReader br = new BufferedReader(new FileReader(filename))) {
            String line;
            while ((line = br.readLine()) != null) {
                words.add(line.toLowerCase(Locale.ROOT));
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private void readFrequencies(String filename) {
        frequencies = new HashMap<>();

        try (BufferedReader br = new BufferedReader(new FileReader(filename))) {
            String line;
            while ((line = br.readLine()) != null) {
                // split line at comma
                String[] s = line.split("[,]", 0);
                // each line will have two parts, first is string, second is integer of frequency
                frequencies.put(s[0], Float.valueOf(s[1]));
            }
        } catch (IOException e) {
            e.printStackTrace();
        }

        // sort
        Collections.sort(words);
    }

    // rank words based on their frequencies
    private List<String> rankWords(List<String> words) {
        List<String> rankedWords = new LinkedList<>();
        Map<Float, String> ranked = new TreeMap<>(); // to sort by natural order
        List<Float> rankingValues = new LinkedList<>();
        for (String word : words) {
            rankingValues.add(frequencies.get(word));
            ranked.put(frequencies.get(word), word);
        }

        Collections.sort(rankingValues);

        for (Float f : rankingValues) {
            String s = ranked.get(f);
            rankedWords.add(s);
        }

        // best words are at the end, need to reverse
        Collections.reverse(rankedWords);

        return rankedWords;

    }



    // checks if all chars in string s1 are in string s2
    private boolean checkString(String s1, String s2) {
        for (int i = 0; i < s1.length(); i++) {
            char c = s1.charAt(i);
            if (s2.indexOf(c) == -1) { // not in string
                return false;
            }
        }

        return true;
    }

    // check the position of a letter in a word
    private boolean checkPos(String s1, String letter, int index) {
        if (letter == "" || letter == null) return false;
        char c = s1.charAt(index);
        if (c == letter.charAt(0)) return true;
        return false;
    }


    private List<String> optimizeLikelyWord(String known, String not, Map<Integer, String> knownInPos, Map<Integer, String> knownButNotInPos) {

        /*
        * known is a string that has all letters known but not in position
        * not is a string that has all the letters known to not be in the word
        * knownInPos stores the position of known letters if user chooses to add them
        * */


        // optimizing for words with only known characters
        List<String> potentialWords = new LinkedList<>();
        for (String word : words) {
            if (checkString(known, word)) {
                potentialWords.add(word);
            }
        }

        // removing words that contain characters known not to be in the word
        List<String> temp = new LinkedList<>();
        temp.addAll(potentialWords);
        for (String word : temp) {
            for (char c : not.toCharArray()) {
                String letter = "" + c;
                if (word.contains(letter)) potentialWords.remove(word);
            }
        }

        // removing words that contain characters in a position we know they are not
        List<String> temp1 = new LinkedList<>();
        temp1.addAll(potentialWords); // stops in-place modification exception for modifying an array we are looping
        for (String word : temp1) {
            for (Integer key : knownButNotInPos.keySet()) {
                String value = knownButNotInPos.get(key);
                for (int i = 0; i < value.length(); i++) {
                    if (word.charAt(key) == (value.charAt(i))) potentialWords.remove(word);
                }
            }
        }

        // optimizing for letters in certain positions
        List<String> toRemove = new LinkedList<>();
        for (String word : potentialWords) {
            for (Integer key: knownInPos.keySet()) {
                String value = knownInPos.get(key);
                if (checkPos(word, value, key) == false) toRemove.add(word);
                else continue;
            }
        }

        for (String word : toRemove) potentialWords.remove(word);

        return potentialWords;

    }

    public void output() {
        Map<Integer, String> knownInPos = new HashMap<>();
        Map<Integer, String> knownButNotInPos = new HashMap<>();
        Scanner scanner = new Scanner(System.in);
        System.out.println("\u001B[106m" + "\u001B[90m" + "Wordle Solver - Current Build: 16-02-22 Finn Lestrange" + "\u001B[0m" + "\n");
        System.out.print("\u001B[103m" + "\u001B[90m" + "Please enter all known letters, regardless of if they are in the correct place, ex. abc (yellow or green letters) :" + "\u001B[0m" + " ");

        String known = scanner.nextLine();
        known = known.toLowerCase(Locale.ROOT);

        // positions of letters in words but in wrong position
        System.out.println();
        System.out.println("\u001B[103m" + "\u001B[90m" + "Please enter the letters that are in the word, but in the wrong position (yellow letters):" + "\u001B[0m" + "");
        for (int i = 0; i < 5; i++) {
            System.out.print("Right letter wrong position (" + (i + 1) + ") : ");
            String line = scanner.nextLine().toLowerCase(Locale.ROOT);
            if (line == "" || line == null || line == "\n") continue;
            else knownButNotInPos.put(i, line);
        }

        System.out.print("\n" + "\u001B[100m" + "\u001B[97m" + "Please input the letters you know are not in the word, ex. hyg (grey letters) :" + "\u001B[0m" + " ");

        String not = scanner.nextLine();
        not = not.toLowerCase(Locale.ROOT);

        System.out.println();
        System.out.println("\u001B[102m" + "\u001B[90m" + "Please input the letters you know the positions of (green letters):" + "\u001B[0m" + " ");

        for (int i = 0; i < 5; i++) {
            System.out.print("Letter in position (" + (i + 1) + ") : ");
            String line = scanner.nextLine().toLowerCase(Locale.ROOT);
            if (line == "" || line == null || line == "\n") continue;
            else knownInPos.put(i, line);
        }

        for (int i = 0; i < knownInPos.size(); i++) if (knownInPos.get(i) == "" || knownInPos.get(i) == null) knownInPos.remove(i);

        System.out.println("\n" + "\u001B[105m" + "\u001B[90m" + "Potential Solutions (ranked based on frequency in the english language)" + "\u001B[0m" + " ");
        List<String> w = optimizeLikelyWord(known, not, knownInPos, knownButNotInPos);
        w = rankWords(w); // ranks the words based on their frequency in the English language
        System.out.println(w + "\n");
    }

    public Solution() {
        // read data
        readWords("words.txt");
        readFrequencies("frequency.txt");

        Scanner s = new Scanner(System.in);

        while (true) {
            output();
            System.out.println();
            System.out.print("Would you like to run the solver again with new letters (y or n)?: ");
            String line = s.nextLine().toLowerCase(Locale.ROOT);
            System.out.println();
            if (line.contains("y")) continue;
            else if (line != "y") {
                break;
            }
        }
    }

    public static void main(String[] args) {
        new Solution();
    }
}

```
