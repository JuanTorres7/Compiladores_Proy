# Compiladores

## Juan Torres Farfán - 201910315
## Neftali Calixto - 

## 1. Comentarios

- Agregar a los token reservados

```c++
class Token {
public:
  enum Type { LPAREN=0, RPAREN, PLUS, MINUS, MULT, DIV, EXP, LT, LTEQ, EQ,  NUM, ID, PRINT, SEMICOLON, COMMA, ASSIGN, CONDEXP, IF, THEN, ELSE, ENDIF, WHILE, DO, ENDWHILE, ERR, END, VAR, NOT , TRUE, FALSE, AND, OR, FOR, COLON, ENDFOR, COMMENT };
  static const char* token_names[36];
  Type type;
  string lexema;
  Token(Type);
  Token(Type, const string source);
};
```

- Modificar el Scanner para poder detectar el simbolo "//" (tomando como base el simbolo "/" de división)

```c++
  ...
  else if (strchr("()+-*/;=<,!:", c)) {
    switch(c) {
    ...
    case '*': 
      c = nextChar();
      if (c == '*') token = new Token(Token::EXP);
      else { rollBack(); token = new Token(Token::MULT); }
      break; 
    /* Modify - Comment (T1) */
    case '/':
      c = nextChar();
      if (c == '/') {
        while (c != '\n' && c != '\0') c = nextChar();
        rollBack();
        token = new Token(Token::COMMENT);
      }
      else if (c == '*') { // New AddOn: Comentario de varias lineas
        bool flag = false;
        while (!flag && c != '\0') {
          c = nextChar();
            if (c == '*') {
              c = nextChar();
              if (c == '/') flag = true;
              else rollBack();
            }
        }
        token = new Token(Token::COMMENT);
      }
      else { token = new Token(Token::DIV); }
      break;
    /* ----- */
  ...
```

- Modificar el Parser para poder realizar las operaciones no fundamentales en el programa. Esto tomando en cuenta que el comentario no es relevante para la ejecución del programa

```c++
StatementList* Parser::parseStatementList() {
  StatementList* p = new StatementList();
  p->add(parseStatement());
  while(match(Token::SEMICOLON)) {
    while (match(Token::COMMENT)) {}
    p->add(parseStatement());
  }
  while (match(Token::COMMENT)) {}
  return p;
}
```


## 2. Generación de código



## 3. Sentencia do-while
- Modificamos agregando las nuevas funciones para la nueva sentencia DoWhile en todas las clases necesarias.
```c++
class TypeVisitor {
public:
  ...
  virtual void visit(DoWhileStatement* e) = 0; // Add - DoWhile (T3)
  ...
};

class ImpVisitor {
public:
  ...
  virtual int visit(DoWhileStatement* e) = 0; /* Add - DoWhile (T3) */
  ...
};

class ImpTypeChecker : public TypeVisitor {
public:
  ImpTypeChecker();
private:
  Environment<ImpType> env;
  ImpType booltype;
  ImpType inttype;

public:
  ...
  void visit(DoWhileStatement*);
  ...
};

class ImpPrinter : public ImpVisitor {
public:
  ...
  int visit(DoWhileStatement*); /* Add - DoWhile (T3) */
  ...
};

class ImpInterpreter : public ImpVisitor {
private:
  Environment<int> env;

public:
  ...
  int visit(DoWhileStatement*);
  ...
};
```

- En el TypeChecker verificamos la correctitud del statement
```c++
void ImpTypeChecker::visit(DoWhileStatement* s) {
  s->body->accept(this);
  if (!s->cond->accept(this).match(booltype)) {
    cout << "Condicional en DoWhile Statement debe de ser: " << booltype << endl;
    exit(0);
  }
  return;
}
```

- Modificamos el Parser para poder detectar la sentencia "Do" y, posteriormente, "While"

```c++
Stm* Parser::parseStatement() {
  ...
  else if (match(Token::DO)) {
    tb = parseBody();
    if (!match(Token::WHILE)) 
      parserError("Esperaba 'While'");
    e = parseExp();
    s = new DoWhileStatement(e, tb);
  }
  ...
  else {
    cout << "No se encontro Statement" << endl;
    exit(0);
  }
  return s;
}
```

- En el interpreter realizamos la lógica de ejecución para la nueva sentencia creada

```c++
int ImpInterpreter::visit(DoWhileStatement* s) {
  do {
    s->body->accept(this);
  } while (s->cond->accept(this));
  return 0;
}
```

- En el printer para mostrar al usuario
```c++
int ImpPrinter::visit(DoWhileStatement* s) {
  cout << "do " << endl;
  s->body->accept(this);
  cout << "while (";
  s->cond->accept(this);
  cout << ")";
  return 0;
}
```

## 4. Sentencia break y continue

## 5. Compilado y prueba de programa

- Con la finalidad de probar las implementaciones de comentario y nueva sentencia DoWhile, ejecutamos lo siguiente:

```c++
g++ -o compile imp_compiler.cpp imp.cpp imp_parser.cpp imp_printer.cpp imp_typechecker.cpp imp_type.cpp imp_interpreter.cpp imp_codegen.cpp
```

- Creamos un archivo (ejemplo_proy.imp) a probar donde se realizan todas las nuevas implementaciones

```
var bool a,b;
var int x,y;
x = 4;
y = 10;
a = x < 10;

// Comentario 1
print(x+y);

if a then
    do
        x=x+2;
        print(x)
    while x<10
else
    print(y*x) // Imprimimos x*y cuando la condicion de x < 10 se incumple
endif

/*
y = 0;
while 0 < x do
  var int z;
  y = y+x;
  x = x-1;
  z=0
endwhile;

print(x+y)
*/
```

```c++
.\compile.exe .\ejemplo_addr.imp
```

