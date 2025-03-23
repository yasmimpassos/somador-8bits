# Somador Completo de 4 Bits

Comecei primeiro desenvolvendo um **meio somador de 1 bit**, para depois fazer um **somador completo de 1 bit**, e por fim construir um **somador de 4 bits completo**.

---

## Meio Somador de 1 Bit

De início, eu comecei fazendo uma **tabela verdade de um meio somador**, na qual tenho como entrada `A` e `B`, e como saída `S` (soma) e `C_out` (o estouro, quando excede um bit).

A tabela verdade ficou dessa forma:

| Entrada A | Entrada B | S | C_out |
|-----------|-----------|---|--------|
|     0     |     0     | 0 |   0    |
|     0     |     1     | 1 |   0    |
|     1     |     0     | 1 |   0    |
|     1     |     1     | 0 |   1    |

Após fazer a tabela verdade, tentei descobrir as relações entre `A` e `B` com `S` e com `C_out`. Reparei que `S` era um **XOR** de `A` e `B`, pois ele só é 1 quando somente um ou outro é 1. Quando ambos são 0 ou ambos são 1, `S` tem como saída 0. Já para o `C_out`, percebi que ele só era 1 se `A` e `B` **conjuntamente** fossem 1, ou seja, uma relação de **AND**.

Ficando dessa forma o circuito do meio somador:

![Meio Somador](/assets/meio_somador.png)

---

## Somador Completo de 1 Bit

Com o meio somador pronto, eu comecei a fazer o somador completo. Para isso, comecei criando uma tabela verdade com as entradas `A`, `B` e `C_in`. Como saída, coloquei `S_1` (a soma considerando apenas `A` e `B`), `C_out1` (carry resultante de `A` e `B`), `S_2` (a soma final considerando também `C_in`) e `C_out2` (carry final).

Fiz isso para tentar entender se tinha como fazer a relação de `S_1` e `C_out1` com o `C_in`, **sem precisar criar uma relação direta com `A` e `B` novamente**. A tabela verdade ficou assim:

| A | B | C_in | S_1 (A⊕B) | C_out1 (A·B) | S_2 (S_1⊕C_in) | C_out2 (C_out1 + (S_1·C_in)) |
|---|---|-------|-------------|---------------|------------------|------------------------------|
| 0 | 0 |   0   |     0       |      0        |        0         |             0                |
| 0 | 0 |   1   |     0       |      0        |        1         |             0                |
| 0 | 1 |   0   |     1       |      0        |        1         |             0                |
| 0 | 1 |   1   |     1       |      0        |        0         |             1                |
| 1 | 0 |   0   |     1       |      0        |        1         |             0                |
| 1 | 0 |   1   |     1       |      0        |        0         |             1                |
| 1 | 1 |   0   |     0       |      1        |        0         |             1                |
| 1 | 1 |   1   |     0       |      1        |        1         |             1                |

Dessa forma, consegui reparar que, da mesma forma que no meio somador, eu poderia utilizar o **XOR** para relacionar `C_in` e `S_1` e assim gerar a saída `S_2`.

Já a parte do `C_out2` foi um pouco mais difícil. Tentei, inicialmente, aplicar o mesmo raciocínio do meio somador, usando **AND** entre `C_in` e `S_1`. Aí percebi que, se `C_out1` for 1 ou o `AND` entre `C_in` e `S_1` for 1, o `C_out2` também será 1. Assim, relacionei ambos com um **OR**, ficando:

```
C_out2 = C_out1 OR (S_1 AND C_in)
```

Com isso, o circuito ficou assim:

![Somador Completo](/assets/somador_completo_1bit.png)

---

Depois de analisar mais a fundo, percebi que o somador completo era **basicamente composto por dois meios somadores**:

- O primeiro somador entre `A` e `B`
- O segundo entre `S_1` e `C_in`
- Por fim, os `C_outs` dos dois meios somadores são conectados com um **OR**

Ficando dessa forma:

![Somador Completo usando meio somador](/assets/somador_completo_1bit_meio_somador.png)

---

## Somador Completo de 4 Bits

Com o somador completo funcionando, fui fazer o somador de 4 bits. E, fazendo ele, percebi que basicamente **eram vários somadores completos em série**.

Imaginando que nosso primeiro número de 4 bits seja `A_0 A_1 A_2 A_3` e o segundo número `B_0 B_1 B_2 B_3`, é só começar pelos bits de menor significância (`A_0` e `B_0`) e seguir conectando o `C_out` de um estágio no `C_in` do próximo.

Isso funciona exatamente como fazemos uma **soma decimal na mão**: quando a soma das unidades ultrapassa 9, levamos 1 para a casa das dezenas. Aqui, quando a soma ultrapassa 1, levamos 1 para o próximo bit.

Então a sequência fica assim:

- `A_0 + B_0`, `C_out` vai para o `C_in` do próximo.
- `A_1 + B_1 + C_in`, e assim por diante.
- Até `A_3 + B_3 + C_in`, cujo `C_out` indica se houve **overflow**.

O circuito ficou assim:

![Somador 4 Bits](/assets/somador_4bits.png)

Além disso:

- Adicionei um **LED** para mostrar visualmente quando ocorre overflow.
- E também coloquei um **display de 7 segmentos**, que mostra a saída da soma em **decimal** (de 0 a 9) e depois continua representando valores em hexadecimal: 10 = A, 11 = B... até F.

Para usar esse display, precisei de um **distribuidor**, que basicamente encaminha a informação de todas as saídas com apenas um cabo.

---

### Código de Teste

Por fim, para testar, utilizei um código de teste que já tinha na biblioteca do Digital, que é esse aqui:

```python
           C_in  A_3 A_2 A_1 A_0  B_3 B_2 B_1 B_0  C_out S_3 S_2 S_1 S_0
        repeat(256)  0    bits(4,n>>4)     bits(4,n)        bits(5,(n>>4)+(n&15))
        repeat(256)  1    bits(4,n>>4)     bits(4,n)        bits(5,(n>>4)+(n&15)+1)
```

Ele simula todas as combinações possíveis entre dois números de 4 bits, com e sem `C_in`.

E o meu circuito passou corretamente!
