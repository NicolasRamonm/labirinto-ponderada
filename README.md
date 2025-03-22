# labirinto-ponderada

## Funcionamento

Quando o robô se move pelo labirinto, ele registra cada movimento na pilha. Esta estrutura permite que ele volte pelo mesmo caminho caso encontre um beco sem saída. A pilha funciona como um registro do caminho percorrido, onde o último movimento é o primeiro a ser desfeito quando necessário.

As chaves encontradas são armazenadas em uma fila. Quando o robô encontra uma porta trancada, ele usa a chave mais antiga da fila para abri-la. Isso faz com que as chaves sejam usadas na ordem em que foram coletadas.

## Estratégia

O robô explora o labirinto tentando se mover em quatro direções: cima, direita, baixo e esquerda. Quando ele não consegue avançar em nenhuma direção, consulta a pilha para voltar e tentar outro caminho.

O sistema mantém um registro dos locais já visitados para evitar que o robô passe pelo mesmo lugar duas vezes, o que poderia causar um loop infinito.

Durante a execução, o programa mostra cada passo do robô:
- Quando encontra uma chave: "Encontrei uma chave na posição (x, y)"
- Quando usa uma chave: "Usando a chave da posição (x, y) para abrir a porta em (a, b)"
- Quando precisa voltar: "Preciso voltar... Retornando para a posição (x, y)"
- Quando chega à saída: "Encontrei a saída! Missão cumprida!"

``` 
class RoboExplorador:
    def __init__(self, labirinto):
        self.labirinto = labirinto
        self.linhas = len(labirinto)
        self.colunas = len(labirinto[0])
        self.pilha_movimentos = []  # Pilha para permitir voltar no caminho (LIFO)
        self.fila_chaves = []       # Fila para guardar as chaves na ordem de coleta (FIFO)
        self.posicao_atual = self.encontrar_inicio()
        self.visitados = set()      # Guarda as células já visitadas para evitar loops

    def encontrar_inicio(self):
        """Localiza onde está o ponto de partida (S) no labirinto."""
        for i in range(self.linhas):
            for j in range(self.colunas):
                if self.labirinto[i][j] == 'S':
                    return (i, j)
        raise ValueError("Não encontrei o ponto de partida (S) no labirinto!")

    def movimento_valido(self, linha, coluna):
        """Verifica se o robô pode se mover para essa posição."""
        # Está dentro dos limites do labirinto?
        if linha < 0 or linha >= self.linhas or coluna < 0 or coluna >= self.colunas:
            return False
        
        # Não é uma parede, né?
        if self.labirinto[linha][coluna] == 'X':
            return False
        
        # Se for uma porta, precisamos ter pelo menos uma chave
        if self.labirinto[linha][coluna] == 'D' and not self.fila_chaves:
            return False
            
        return True

    def mover(self, linha, coluna):
        """Movimenta o robô para uma nova posição se for possível."""
        if not self.movimento_valido(linha, coluna):
            return False
        
        # Guarda a posição anterior na pilha para poder voltar se necessário
        self.pilha_movimentos.append(self.posicao_atual)
        
        # Atualiza para a nova posição
        self.posicao_atual = (linha, coluna)
        linha_atual, coluna_atual = self.posicao_atual
        celula_atual = self.labirinto[linha_atual][coluna_atual]
        
        # Verifica o que tem na célula atual
        if celula_atual == 'K':
            self.fila_chaves.append((linha_atual, coluna_atual))
            print(f"Opa! Encontrei uma chave na posição ({linha_atual}, {coluna_atual})")
        
        elif celula_atual == 'D':
            if self.fila_chaves:
                chave_usada = self.fila_chaves.pop(0)  # Usa a chave mais antiga primeiro
                print(f"Usando a chave da posição {chave_usada} para abrir a porta em ({linha_atual}, {coluna_atual})")
            else:
                # Não deveria chegar aqui por causa da verificação em movimento_valido
                print("Erro: Tentei abrir uma porta sem ter chave!")
                return False
        
        elif celula_atual == 'E':
            print(f"Encontrei a saída em ({linha_atual}, {coluna_atual})! Missão cumprida!")
            return True
        
        # Marca como visitado para não passar aqui de novo
        self.visitados.add(self.posicao_atual)
        print(f"Avançando para a posição ({linha_atual}, {coluna_atual})")
        return True

    def voltar(self):
        """Volta para a posição anterior usando a pilha de movimentos."""
        if not self.pilha_movimentos:
            print("Estou preso! Não consigo voltar mais.")
            return False
        
        posicao_anterior = self.pilha_movimentos.pop()  # Remove o último movimento da pilha
        self.posicao_atual = posicao_anterior
        linha, coluna = self.posicao_atual
        
        print(f"Preciso voltar... Retornando para a posição ({linha}, {coluna})")
        return True

    def explorar_labirinto(self):
        """Explora o labirinto até encontrar a saída ou concluir que não há solução."""
        print(f"Iniciando exploração na posição {self.posicao_atual}")
        
        # Direções que o robô pode tentar: cima, direita, baixo, esquerda
        direcoes = [(-1, 0), (0, 1), (1, 0), (0, -1)]
        nomes_direcoes = ["cima", "direita", "baixo", "esquerda"]
        
        while True:
            linha_atual, coluna_atual = self.posicao_atual
            
            # Chegamos ao destino?
            if self.labirinto[linha_atual][coluna_atual] == 'E':
                print("Consegui! Cheguei à saída do labirinto!")
                return True
            
            # Tenta se mover em alguma direção
            movimento_feito = False
            for i, (dl, dc) in enumerate(direcoes):
                nova_linha, nova_coluna = linha_atual + dl, coluna_atual + dc
                nova_posicao = (nova_linha, nova_coluna)
                
                # Se puder ir para essa direção e não visitou ainda
                if self.movimento_valido(nova_linha, nova_coluna) and nova_posicao not in self.visitados:
                    print(f"Vou tentar ir para {nomes_direcoes[i]}")
                    if self.mover(nova_linha, nova_coluna):
                        movimento_feito = True
                        
                        # Se chegou na saída, termina
                        if self.labirinto[nova_linha][nova_coluna] == 'E':
                            return True
                        break
            
            # Se não conseguiu ir para nenhuma direção, precisa voltar
            if not movimento_feito:
                print("Parece que estou em um beco sem saída. Preciso voltar...")
                if not self.voltar():
                    print("Não consigo mais voltar. Este labirinto não tem solução!")
                    return False


def criar_labirinto_do_exemplo():
    """Cria o labirinto conforme o exemplo do enunciado."""
    labirinto = [
        ['S', '.', 'K', '.', 'D', '.', 'E'],
        ['.', 'X', '.', 'X', '.', 'X', '.'],
        ['.', '.', 'K', '.', 'D', '.', '.']
    ]
    return labirinto


def imprimir_labirinto(labirinto):
    """Mostra o labirinto de forma visual na tela."""
    for linha in labirinto:
        print(' '.join(linha))
    print()


def main():
    # Prepara o labirinto do exemplo
    labirinto = criar_labirinto_do_exemplo()
    
    print("Labirinto a ser explorado:")
    imprimir_labirinto(labirinto)
    
    # Cria o robô e inicia a exploração
    robo = RoboExplorador(labirinto)
    robo.explorar_labirinto()
    
    # Mostra resultados finais
    print("\nResumo da exploração:")
    print(f"Total de movimentos realizados: {len(robo.pilha_movimentos)}")
    print(f"Chaves não utilizadas: {len(robo.fila_chaves)}")


if __name__ == "__main__":
    main()
```
