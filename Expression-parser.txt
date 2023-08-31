/* 
This module contains a simple expression parser that does not recognize variables.
*/

#include<stdio.h>
#include<ctype.h>
#include<stdlib.h>
#include<string.h>

#define DELIMETER 1
#define VARIABLE  2
#define NUMBER    3

char *prog;
char token[80];
char tok_type;

void eval_exp(double *answer), eval_exp1(double *answer);
void eval_exp3(double *answer), eval_exp4(double *answer);
void eval_exp5(double *answer), eval_exp6(double *answer);
void atom(double *answer);
void get_token(void);
void serror(int error);
int isdelim(char c);

/* Parser entry point */
void eval_exp(double *answer)
{
 get_token();
 if(!*token)
 {
  serror(2);
  return;
 }
 eval_exp2(answer);

 if(*token) serror(0); /* Last tokken must be null */
}

/* Add or subtract two terms */
void eval_exp2(double *answer)
{
 register char op;
 double temp;
 
 eval_exp3(answer);
 while((op = *token) == '+' || op == '-')
 {
  get_token();
  eval_exp3(&temp);
  switch(op)
  {
   case '+':
    *answer += temp;
    break;
   case '-':
    *answer -= temp;
    break;
  }
 }
}

/* Multiply or divide or modulus of two factors */
void eval_exp3(double *answer)
{
 register char op;
 double temp;

 eval_exp4(answer);
 while((op = *token) == '*' || op == '/' || op == '%')
 {
  get_token();
  eval_exp4(answer);
  switch(op)
  {
   case '*':
    *answer *= temp;
    break;
   case '/':
    *answer /= temp;
    break;
   case '%':
    *answer %= temp;
    break;
  }
 }
}
/* Exponential processing */
void eval_exp4(double *answer)
{
 double temp, ex;
 register int t;

 eval_exp5(answer);
 
 if(*token == '^')
 {
  get_token();
  eval_exp4(&temp);
  ex = *answer;
  if(temp == 0.0)
  {
   *answer = 1.0;
   return;
  }
  for(t = temp-1; t>0; --t)
  {
   *answer *= (double)ex;
  }
 }
}

/* Unary + or - */
void eval_exp5(double *answer)
{
 register char op;
 op = 0;
 if((tok_type == DELIMETER) && *token == '+' || *token == '-')
 {
  op = *token;
  get_token();
 }
 
 eval_exp6(answer);
 if(op == '-') *answer = -(*answer);
}

/* Paranthesis processing */
void eval_exp6(double *answer)
{
 if((*token == '('))
 {
  get_token();
  eval_exp2(answer);
  if(*token != ')')
  {
   serror(1);
  }
  get_token();
 }
 else
  atom(answer);
}

/* Extracting value of number */
void atom(double *answer)
{
 if(tok_type == NUMBER)
 {
  *answer = atof(*token);
  get_token();
  return;
 }
 serror(0);
}

/* Returning token to input stream */
void putback(void)
{
 char *t;
 t = token;
 for(; *t; t++) prog--;
}

/* Syntax errors displaying */
void serror(int error)
{
 static char *e[] = {
     "Syntax Error",
     "Unbalanced Pranthesis",
     "No Expression Present",
     "Division By Zero"
 };
 printf("%s\n", e[error]);
}

/* Getting next token */
void get_token(void)
{
 register char *temp;
 tok_type = 0;
 temp = token;
 *temp = '\0';

 if(!*prog) return;
 while(isspace(*prog) ++prog;
 
 if(strchr("+-*/%^=()", *prog))
 {
  tok_type = DELIMETER;
  *temp++ = *prog++;
 }
 else if(isalpha(*prog))
 {
  while(!isdelim(*prog)) *temp++ = *prog++;
  tok_type = VARIABLE;
 }
 
 else if(isdigit(*prog))
 {
  while(!isdelim(*prog)) *temp++ = *prog++;
  tok_type = NUMBER;
 }
 *temp = '\0';
}

/* Checking delimeter */
int isdelim(char c)
{
 if(strchr("+-*/%^=()", c) || c == 9 || c == '\r' || c == 0)
 {
  return 1;
 }
 return 0;
}

/* Main function */ 
int main(void)
{
 double answer;
 char *p;
 
 p = (char *) malloc(100);
 if(!p)
 {
  printf("ERROR: Allocation not occured.\n");
  exit(1);
 }
 
/* Processing the expressions from input stream until a blank line is entered */
do {
 prog = p;
 printf("Enter the expression: ");
 gets(prog);
 if(!*prog) break;
 eval_exp(&answer);
 printf("Answer: %.2f\n", answer);
 }while(*p);
 return 0;
}