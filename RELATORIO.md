# 📝 Relatório do Laboratório 2 - Chamadas de Sistema

---

## 1️⃣ Exercício 1a - Observação printf() vs 1b - write()

### 💻 Comandos executados:
```bash
strace -e write ./ex1a_printf
strace -e write ./ex1b_write
```

### 🔍 Análise

**1. Quantas syscalls write() cada programa gerou?**
- ex1a_printf: 9 syscalls
- ex1b_write: 7 syscalls

**2. Por que há diferença entre os dois métodos? Consulte o docs/printf_vs_write.md**

O printf() é uma função da linguagem que chama o write() para exibir algo na tela.
Já o write() é uma chamada do sistema no Linux, que se conecta com o kernel para realizar a impressão de fato.

**3. Qual método é mais previsível? Por quê você acha isso?**

O printf() usa um buffer para juntar várias chamadas de write(), o que o torna mais eficiente.
Ele só faz a chamada ao sistema quando o buffer está cheio ou quando é forçado a isso.

---

## 2️⃣ Exercício 2 - Leitura de Arquivo

### 📊 Resultados da execução:
- File descriptor: 3
- Bytes lidos: 127

### 🔧 Comando strace:
```bash
strace -e openat,read,close ./ex2_leitura
```

### 🔍 Análise

**1. Qual file descriptor foi usado? Por que não começou em 0, 1 ou 2?**

Os registradores 0, 1 e 2 são usados para as entradas e saídas padrão (0 = stdin, 1 = stdout, 2 = stderr).
Nesse caso, o descritor retornado foi o 3, que é o primeiro disponível para o processo usar.

**2. Como você sabe que o arquivo foi lido completamente?**

A variável bytes_lidos nunca recebeu um valor menor que 0, ou seja, não houve erro durante o processo, indicando que ele foi concluído corretamente e sem falhas.

**3. Por que verificar retorno de cada syscall?**

Para garantir que a chamada foi executada corretamente e identificar possíveis erros durante o processo.

---

## 3️⃣ Exercício 3 - Contador com Loop

### 📋 Resultados (BUFFER_SIZE = 64):
- Linhas: 25 (esperado: 25)
- Caracteres: 1300
- Chamadas read(): 21
- Tempo: 0.000135 segundos

### 🧪 Experimentos com buffer:

| Buffer Size | Chamadas read() | Tempo (s) |
|-------------|-----------------|-----------|
| 16          |82               |0.000250   |
| 64          |21               |0.000135   |
| 256         |6                |0.000108   |
| 1024        |2                |0.000095   |

### 🔍 Análise

**1. Como o tamanho do buffer afeta o número de syscalls?**

Com um buffer maior, o printf consegue juntar mais conteúdo antes de chamar o write(), fazendo menos chamadas ao sistema.

**2. Todas as chamadas read() retornaram BUFFER_SIZE bytes? Discorra brevemente sobre**

Quando a variável bytes_lidos recebe 0 do read, isso indica que o arquivo chegou ao fim e não há mais conteúdo para ler.

**3. Qual é a relação entre syscalls e performance?**

Quanto mais syscalls um programa faz, mais vezes ele precisa interagir com o kernel, o que gera custo extra.
Reduzir a quantidade de chamadas, como usando buffers, melhora a performance.

---

## 4️⃣ Exercício 4 - Cópia de Arquivo

### 📈 Resultados:
- Bytes copiados: 1364
- Operações: 6
- Tempo: 0.000259 segundos
- Throughput: 5142.98 KB/s

### ✅ Verificação:
```bash
diff dados/origem.txt dados/destino.txt
```
Resultado: [ ] Idênticos [ ] Diferentes

### 🔍 Análise

**1. Por que devemos verificar que bytes_escritos == bytes_lidos?**

Para garantir que o arquivo foi copiado completamente e sem erros.

**2. Que flags são essenciais no open() do destino?**

O_CREAT: cria o arquivo se ele não existir.
O_WRONLY: abre o arquivo apenas para escrita.
O_TRUNC: apaga o conteúdo do arquivo se ele já existir.

**3. O número de reads e writes é igual? Por quê?**

O número de read() e write() é igual, pois cada leitura é seguida de uma escrita.

**4. Como você saberia se o disco ficou cheio?**

Dando erro na chamada write(), que retornaria -1 e definiria errno como ENOSPC (sem espaço no dispositivo).

**5. O que acontece se esquecer de fechar os arquivos?**

Os arquivos permaneceriam abertos, consumindo recursos do sistema, e os dados podem não ser completamente gravados no disco.

---

## 🎯 Análise Geral

### 📖 Conceitos Fundamentais

**1. Como as syscalls demonstram a transição usuário → kernel?**

A própria chamada já mostra a transição, pois a syscall atua como ponte entre o pedido do usuário e o kernel, que executa a ação necessária.

**2. Qual é o seu entendimento sobre a importância dos file descriptors?**

```
Informa ao sistema qual arquivo deve ser usado, recebendo em troca um número de descritor (fd) para identificá-lo.
```

**3. Discorra sobre a relação entre o tamanho do buffer e performance:**

```
Um buffer maior permite acumular mais dados antes de chamar o sistema, reduzindo trocas entre modo usuário e kernel e economizando tempo.
Porém, buffers maiores exigem mais memória.
```

### ⚡ Comparação de Performance

```bash
# Teste seu programa vs cp do sistema
time ./ex4_copia
time cp dados/origem.txt dados/destino_cp.txt
```

**Qual foi mais rápido?** _____

**Por que você acha que foi mais rápido?**

```
O sistema foi mais rápido, pois teve 0m0.003s de sys e 0m0.000s de user, enquanto o outro teve 0m0.002s em cada.
Provavelmente foi mais rápido por não precisar alternar entre modos, já que o tempo em user é zero.
```

---

## 📤 Entrega
Certifique-se de ter:
- [ ] Todos os códigos com TODOs completados
- [ ] Traces salvos em `traces/`
- [ ] Este relatório preenchido como `RELATORIO.md`

```bash
strace -e write -o traces/ex1a_trace.txt ./ex1a_printf
strace -e write -o traces/ex1b_trace.txt ./ex1b_write
strace -o traces/ex2_trace.txt ./ex2_leitura
strace -c -o traces/ex3_stats.txt ./ex3_contador
strace -o traces/ex4_trace.txt ./ex4_copia
```
# Bom trabalho!
