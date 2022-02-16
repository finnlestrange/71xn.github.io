---
layout: single
title: "Wordle Solver / Solution Finder"
date: 2022-02-16 08:49:23 +0100
categories: programming
excerpt: ✏ Java 17 Wordle Solver / Solution Finder 
---

# ✏ Java 17 Wordle Solver / Solution Finder

### [GitHub Link](https://github.com/71xn/algorithmsDataStructures/blob/main/wordleSolver/src/tech/finnlestrange/Solution.java)

### [Wordle Link](https://www.nytimes.com/games/wordle/index.html)

## Description

> This code aims to find a list of potential solutions to a wordle problem, based on the known letters and the known letters in relative positions.

### TODO

* Still to do is to add in functionality such that potential solutions are ranked based on their English language frequency to find more optimal solutions.

## Demo

![](/images/programming/wordle.png)

## Code

```java
package tech.finnlestrange;

import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
import java.util.*;

public class Solution {

    List<String> words;

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


    private List<String> optimizeLikelyWord(String known, String not, Map<Integer, String> knownInPos) {

        /*
        * known is a string that has all letters known but not in position
        * not is a string that has all the letters known to not be in the word
        * knownInPos stores the position of known letters if user chooses to add them
        * */

        readWords("words.txt");

        Collections.sort(words);

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
        Scanner scanner = new Scanner(System.in);
        System.out.println("Wordle Solver: ");
        System.out.print("\nPlease input the letters you know with no spaces: ");

        String known = scanner.nextLine();
        known = known.toLowerCase(Locale.ROOT);

        System.out.print("\nPlease input the letters you know are not in the word with no spaces: ");

        String not = scanner.nextLine();
        not = not.toLowerCase(Locale.ROOT);

        System.out.println("Please input the letters you know the positions of: ");

        for (int i = 0; i < 5; i++) {
            System.out.print("Letter in position (" + (i + 1) + ") : ");
            String line = scanner.nextLine().toLowerCase(Locale.ROOT);
            if (line == "" || line == null || line == "\n") continue;
            else knownInPos.put(i, line);
        }

        for (int i = 0; i < knownInPos.size(); i++) if (knownInPos.get(i) == "" || knownInPos.get(i) == null) knownInPos.remove(i);

        System.out.println("\nPotential Solutions: ");
        List<String> w = optimizeLikelyWord(known, not, knownInPos);
        System.out.println(w);

    }

    public Solution() {
        while (true) {
            output();
            System.out.println();
            System.out.println("Would you like to run the code again with new letters: y or n");
            Scanner s = new Scanner(System.in);
            String line = s.nextLine().toLowerCase(Locale.ROOT);
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