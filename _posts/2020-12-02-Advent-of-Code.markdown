---
layout: single
title:  "Advent of Code 2020 Solution Writeups"
date:   2020-09-18 08:49:23 +0100
categories: programming
excerpt: My solutions to the 2020 advent of code
---

[Advent of Code](https://adventofcode.com/)

## Day 1

### Challenge 1

{ % highlight java linenos % }
import java.util.ArrayList;
import java.util.Scanner;

public class Day1_1 {
	
	public static void main(String[] args) {
		
		Scanner scanner = new Scanner(System.in);
		ArrayList<Integer> expenses = new ArrayList<>(); 
		
		boolean done = false;
		while (!done) {
			String temp = scanner.next();
			if (!temp.equals("end")) {
				expenses.add(Integer.parseInt(temp));
			} else {
				done = true;
			}
		}
		
		int num1 = 0;
		int num2 = 0;
		int temp1 = 0;
		int temp2 = 0;
		int sum = 0;
		boolean found = false;
		while (!found) {
			for (int i = 0; i < expenses.size(); i++) {
				for (int x = 0; x < expenses.size(); x++) {
					temp1 = expenses.get(i);
					temp2 = expenses.get(x);
					sum = temp1 + temp2;
					if (sum == 2020) {
						found = true;
						num1 = expenses.get(i);
						num2 = expenses.get(x);
					}
				}
			}
		}
		
		System.out.println(num1 * num2);
	}
}

{ % endhighlight % }

### Challenge 2

{ % highlight java linenos % }
import java.util.ArrayList;
import java.util.Scanner;

public class Day1_2 {
	
	public static void main(String[] args) {
		
		Scanner scanner = new Scanner(System.in);
		ArrayList<Integer> expenses = new ArrayList<>(); 
		
		boolean done = false;
		while (!done) {
			String temp = scanner.next();
			if (!temp.equals("end")) {
				expenses.add(Integer.parseInt(temp));
			} else {
				done = true;
			}
		}
		

		boolean found = false;
		while (!found) {
			for (int i = 0; i < expenses.size(); i++) {
				for (int x = 0; x < expenses.size(); x++) {
					for (int y = 0; y < expenses.size(); y++) {

						if ((expenses.get(i) + expenses.get(x) + expenses.get(y)) == 2020) {
							found = true;
							System.out.println(expenses.get(i) * expenses.get(x) * expenses.get(y));
							break;
						}
					}
				}
			}
		}	
	}
}

{ % endhighlight % }


## Day 2

### Challenge 1

{ % highlight java linenos % }
import java.util.Scanner;
import java.util.regex.Pattern; 

public class Day2_1 {
	public static void main(String[] args) {
		Scanner scanner = new Scanner(System.in);
		String num = "";
		int number1 = 0;
		int number2 = 0;
		char character = 'a';
		String password = "";
		int counter = 0;
		int count = 0;
		char tempCH = 'a';
		
		while (scanner.hasNext()) {
			num = scanner.next();
			if (num.length() == 3) {
				number1 = Integer.parseInt(String.valueOf(num.charAt(0)));
				number2 = Integer.parseInt(String.valueOf(num.charAt(2)));
			} else if (num.length() == 4) {
				number1 = Integer.parseInt(String.valueOf(num.charAt(0)));
				number2 = Integer.parseInt(String.valueOf(num.charAt(2)) + String.valueOf(num.charAt(3)));
			} else if (num.length() == 5) {
				number1 = Integer.parseInt(String.valueOf(num.charAt(0)) + String.valueOf(num.charAt(1)));
				number2 = Integer.parseInt(String.valueOf(num.charAt(3)) + String.valueOf(num.charAt(4)));
			}

			character = scanner.next().charAt(0);
			password = scanner.next();
			for (int i = 0; i < password.length(); i++) {
				tempCH = password.charAt(i);
				if (tempCH == character) {
					count++;
				}
			}
			
			if ((count >= number1) && (count <= number2)) {
				counter++;
			}
			count = 0;
			System.out.println(counter);
		}
		scanner.close();
	}
}
{ % endhighlight % }


### Challenge 2

{ % highlight java linenos % }
import java.util.Scanner;

public class Day2_2 {
	
	public static boolean passCheck(String p, int n, int m, char c) {
		if (((p.charAt(n) == c) && (p.charAt(m) != c)) || ((p.charAt(n) != c) && (p.charAt(m) == c))) {
			return true;
		} else {
			return false;
		}
	}
	
	public static void main(String[] args) {
		Scanner scanner = new Scanner(System.in);
		String num = "";
		int number1 = 0;
		int number2 = 0;
		char character = 'a';
		String password = "";
		int counter = 0;

		while (scanner.hasNext()) {
			num = scanner.next();
			if (num.length() == 3) {
				number1 = Integer.parseInt(String.valueOf(num.charAt(0))) -1;
				number2 = Integer.parseInt(String.valueOf(num.charAt(2))) -1;
			} else if (num.length() == 4) {
				number1 = Integer.parseInt(String.valueOf(num.charAt(0))) -1;
				number2 = Integer.parseInt(String.valueOf(num.charAt(2)) + String.valueOf(num.charAt(3))) -1;
			} else if (num.length() == 5) {
				number1 = Integer.parseInt(String.valueOf(num.charAt(0)) + String.valueOf(num.charAt(1))) -1;
				number2 = Integer.parseInt(String.valueOf(num.charAt(3)) + String.valueOf(num.charAt(4))) -1;
			}
			character = scanner.next().charAt(0);
			password = scanner.next();
			if (passCheck(password, number1, number2, character)) {
				counter++;
			}
			
			System.out.println(counter);
		}
		scanner.close();
	}
}

{ % endhighlight % }