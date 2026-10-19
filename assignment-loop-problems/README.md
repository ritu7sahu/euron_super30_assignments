#Python loop and loop control
problem 1 : Print numbers from 1–100 but skip numbers divisible by 5 using continue.
Logic: Iterating for loop and using print getting numbers and checking condition whether it is divisible by 5 or not.If number is divisible by 5 we are skipping it using continue.

problem2 : Iterate from 1–100 and stop when you encounter the first number divisible by both 7 and 11.
Logic:  Iterating for loop and using print getting numbers and checking condition whether it is divisible by 7 and 11 both.If number is divisible by  7 and 11 both.we are terminating the loop using break and priting the no till that.

problem 3: search for a user-provided number inside a list. Use for-else to print:
#"Number Found"
#"Number Not Found"
Logic: Take input from user and create a list. Iterate to the list and check whether number equals to list element, if no found, we are printing no found else go to the else block of for.
key concept:
Match found → break → skip else
No match → loop finishes → execute else

problem 4: use enumerate() to display
 1 Aman
 2 Ravi
 3 Sudhanshu
Logic: 
1. enumerate(names, start=1) returns each item's index and value, starting the index at 1.
2. index stores the current number.
3. value stores the current name.
4. The f-string f"{index} {value}" prints the number and name together.

problem 5:Print the following pattern
 *
 **
 ***
 ****
 *****
Logic: This program uses nested for loops to print a right-angled triangle star pattern. The outer loop controls the rows, while the inner loop prints stars equal to the current row number. end="" keeps stars on the same line, and print() moves to the next line.

problem 6:  Print
*****
****
***
**     
*
Logic : This program uses nested for loops to print an inverted right-angled triangle star pattern. The outer loop decreases the row count from 5 to 0, while the inner loop prints fewer stars in each row. end="" keeps stars on the same line, and print() moves to the next line.

problem 7: Generate multiplication tables from 1 to 10 using nested loops.
Logic: The outer loop iterates from 1 to 10 to select each multiplication table. The inner loop multiplies that number by 1 to 10 and prints the results. An extra print() adds a blank line between tables.

problem 8:Find all numbers between 1 and 200 divisible by both 3 and 5
Logic : The program iterates through numbers from 1 to 200 using a for loop. It checks whether each number is divisible by both 3 and 5 using the modulo (%) operator and prints the numbers that satisfy both conditions.

problem 9: given a list containing duplicate elements, create another list containing only unique elements without using set().
Logic : The program iterates through the list and checks whether each element already exists in unique_list using count(). If the element is already present, continue skips it; otherwise, the element is appended to unique_list.

problem 10:count : positive numbers,negative numbers,zeros
Logic : The program iterates through a list and uses conditional statements to classify each number as positive, negative, or zero. It increments the corresponding counter and displays the final counts.

problem 11:Write a program to determine whether a number is prime using a loop.
Logic : The for-else block prints "Prime number" only if the loop finishes without encountering break. If any number divides num1 evenly, it prints "Not a prime number" and stops the loop.

problem12 : Print all prime numbers between 1 and 100.
Logic: - The outer loop checks numbers from 1 to 100.
- The inner loop checks whether each number is divisible by any number between 1 and i - 1.
- If a divisor is found, break stops the inner loop.
- The for-else block prints the number only when no divisor is found.


