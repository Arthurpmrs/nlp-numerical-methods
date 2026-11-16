# Questão 2.c — Forma aninhada (Horner) do polinômio de Maclaurin

## De onde vem a forma aninhada

Partimos do polinômio (item a):

$$P_8(x) = 1 - \frac{x^2}{2} + \frac{x^4}{24} - \frac{x^6}{720} + \frac{x^8}{40320}$$

**Passo 1 — trocar variável.** Como só aparecem potências pares de $x$, fazemos $u = x^2$. Isso transforma o polinômio de grau 8 em $x$ num polinômio de grau 4 em $u$:

$$P_8 = \underbrace{1}_{c_0} + \underbrace{\left(-\tfrac12\right)}_{c_1}u + \underbrace{\tfrac{1}{24}}_{c_2}u^2 + \underbrace{\left(-\tfrac1{720}\right)}_{c_3}u^3 + \underbrace{\tfrac1{40320}}_{c_4}u^4$$

**Passo 2 — fatorar $u$ progressivamente (Horner).** A ideia do método de Horner é: em vez de calcular $u, u^2, u^3, u^4$ separadamente, colocar $u$ em evidência de fora para dentro:

$$c_0 + c_1u + c_2u^2 + c_3u^3 + c_4u^4 = c_0 + u\Big(c_1 + c_2u + c_3u^2 + c_4u^3\Big)$$

Repetindo o mesmo truque dentro do parêntese:

$$= c_0 + u\Big(c_1 + u\big(c_2 + c_3u + c_4u^2\big)\Big) = c_0 + u\Big(c_1 + u\big(c_2 + u(c_3 + c_4u)\big)\Big)$$

Substituindo os coeficientes, chegamos exatamente na fórmula usada no notebook:

$$P_8(x) = 1 + u\left(-\frac12 + u\left(\frac1{24} + u\left(-\frac1{720} + u\cdot\frac1{40320}\right)\right)\right)$$

## Por que isso é "aninhado"

Repare que $u$ nunca é elevado a uma potência explícita — a cada nível você só faz **uma multiplicação por $u$** e **uma soma com o próximo coeficiente**, reaproveitando o resultado acumulado do nível anterior. É por isso que, no código (célula do item d), calculamos de dentro para fora:

```python
u = x * x                       # u = x²  (1 mult.)
p = coefs[4]                    # começa com o coeficiente mais interno (1/40320)
p = u * p; p = p + coefs[3]     # nível: (-1/720 + u·1/40320)
p = u * p; p = p + coefs[2]     # nível: (1/24 + u·(...))
p = u * p; p = p + coefs[1]     # nível: (-1/2 + u·(...))
p = u * p; soma = p + coefs[0]  # nível final: 1 + u·(...)
```

Cada linha corresponde a "descascar" um parêntese da fórmula, de dentro para fora.

## Por que isso economiza operações

- **Sem aninhamento:** você precisa calcular $x^2, x^4, x^6, x^8$ como multiplicações separadas (4 mult.), multiplicar cada uma pelo seu coeficiente (4 mult.) e somar tudo (4 somas) → **12 operações**.
- **Aninhado:** você calcula $u=x^2$ uma vez (1 mult.) e depois só alterna "multiplica por $u$, soma coeficiente" 4 vezes (4 mult. + 4 somas) → **9 operações**.

A economia vem de nunca recalcular potências de $u$ do zero — cada multiplicação por $u$ já "carrega" o grau anterior embutido no valor acumulado `p`. Como as duas expressões são algebricamente idênticas, o resultado final ($0{,}8776$) é o mesmo — a diferença é só a quantidade de trabalho aritmético para chegar lá.
