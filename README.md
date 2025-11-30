# Desafio-Free-Fire
typedef enum { ALIMENTO=0, FERRAMENTA=1, ARMA=2, MATERIAL=3 } TipoItem;

const char *tipo_texto[] = {"Alimento","Ferramenta","Arma","Material"};

/* ---------- Item e inventário (vetor) ---------- */
typedef struct {
    char nome[NOME_MAX];
    TipoItem tipo;
    int prioridade; // 1..10 (maior = mais importante)
    int quantidade;
} Item;

typedef struct {
    Item *itens;
    int tamanho;
    int capacidade;
} MochilaVetor;

void init_mochila_v(MochilaVetor *m, int cap) {
    m->itens = malloc(sizeof(Item)*cap);
    m->tamanho = 0;
    m->capacidade = cap;
}

void ensure_cap_v(MochilaVetor *m) {
    if (m->tamanho >= m->capacidade) {
        m->capacidade *= 2;
        m->itens = realloc(m->itens, sizeof(Item)*m->capacidade);
    }
}

void adicionar_item_v(MochilaVetor *m, const char *nome, TipoItem tipo, int pri, int qtd) {
    // se já existe mesmo nome e tipo, soma quantidade
    for (int i=0;i<m->tamanho;i++){
        if (strcmp(m->itens[i].nome,nome)==0 && m->itens[i].tipo==tipo) {
            m->itens[i].quantidade += qtd;
            return;
        }
    }
    ensure_cap_v(m);
    strncpy(m->itens[m->tamanho].nome, nome, NOME_MAX-1);
    m->itens[m->tamanho].nome[NOME_MAX-1]=0;
    m->itens[m->tamanho].tipo = tipo;
    m->itens[m->tamanho].prioridade = pri;
    m->itens[m->tamanho].quantidade = qtd;
    m->tamanho++;
}

int remover_item_v(MochilaVetor *m, const char *nome, TipoItem tipo, int qtd) {
    for (int i=0;i<m->tamanho;i++){
        if (strcmp(m->itens[i].nome,nome)==0 && m->itens[i].tipo==tipo) {
            if (m->itens[i].quantidade < qtd) return 0;
            m->itens[i].quantidade -= qtd;
            if (m->itens[i].quantidade == 0) {
                // remove elemento deslocando
                for (int j=i;j<m->tamanho-1;j++) m->itens[j]=m->itens[j+1];
                m->tamanho--;
            }
            return 1;
        }
    }
    return 0;
}

void listar_mochila_v(MochilaVetor *m) {
    printf("\n--- Mochila (vetor) ---\n");
    if (m->tamanho==0){ printf("Vazia\n"); return; }
    for (int i=0;i<m->tamanho;i++){
        printf("[%d] %s | %s | pri=%d | qtd=%d\n", i,
               m->itens[i].nome, tipo_texto[m->itens[i].tipo],
               m->itens[i].prioridade, m->itens[i].quantidade);
    }
}

/* ---------- Lista encadeada (para comparação) ---------- */
typedef struct Node {
    Item item;
    struct Node *next;
} Node;

typedef struct {
    Node *head;
    int tamanho;
} MochilaLista;

void init_mochila_l(MochilaLista *ml) { ml->head = NULL; ml->tamanho=0; }

void adicionar_item_l(MochilaLista *ml, const char *nome, TipoItem tipo, int pri, int qtd) {
    // procura se existe
    Node *p = ml->head;
    while(p) {
        if (strcmp(p->item.nome,nome)==0 && p->item.tipo==tipo) {
            p->item.quantidade += qtd; return;
        }
        p = p->next;
    }
    Node *n = malloc(sizeof(Node));
    strncpy(n->item.nome,nome,NOME_MAX-1); n->item.nome[NOME_MAX-1]=0;
    n->item.tipo = tipo; n->item.prioridade = pri; n->item.quantidade = qtd;
    n->next = ml->head; ml->head = n; ml->tamanho++;
}

int remover_item_l(MochilaLista *ml, const char *nome, TipoItem tipo, int qtd) {
    Node *p = ml->head, *prev = NULL;
    while(p) {
        if (strcmp(p->item.nome,nome)==0 && p->item.tipo==tipo) {
            if (p->item.quantidade < qtd) return 0;
            p->item.quantidade -= qtd;
            if (p->item.quantidade==0) {
                if (prev) prev->next = p->next; else ml->head = p->next;
                free(p); ml->tamanho--;
            }
            return 1;
        }
        prev = p; p = p->next;
    }
    return 0;
}

void listar_mochila_l(MochilaLista *ml) {
    printf("\n--- Mochila (lista encadeada) ---\n");
    if (!ml->head) { printf("Vazia\n"); return; }
    Node *p = ml->head; int i=0;
    while(p) {
        printf("[%d] %s | %s | pri=%d | qtd=%d\n", i, p->item.nome, tipo_texto[p->item.tipo],
               p->item.prioridade, p->item.quantidade);
        p = p->next; i++;
    }
}

/* ---------- Ordenação e busca ---------- */
/* Bubble (simples) - para comparação */
void bubble_sort_nome(Item *a, int n) {
    for (int i=0;i<n-1;i++)
        for (int j=0;j<n-1-i;j++)
            if (strcmp(a[j].nome, a[j+1].nome) > 0) {
                Item tmp = a[j]; a[j] = a[j+1]; a[j+1] = tmp;
            }
}

/* Selection sort por prioridade (decrescente) */
void selection_sort_prioridade(Item *a, int n) {
    for (int i=0;i<n-1;i++){
        int sel = i;
        for (int j=i+1;j<n;j++)
            if (a[j].prioridade > a[sel].prioridade) sel = j;
        if (sel != i) { Item tmp = a[i]; a[i] = a[sel]; a[sel] = tmp; }
    }
}

/* Ordena por tipo (inteiro) e depois por nome */
void selection_sort_tipo_nome(Item *a, int n) {
    for (int i=0;i<n-1;i++){
        int sel = i;
        for (int j=i+1;j<n;j++){
            if (a[j].tipo < a[sel].tipo) sel = j;
            else if (a[j].tipo == a[sel].tipo && strcmp(a[j].nome,a[sel].nome) < 0) sel = j;
        }
        if (sel != i) { Item tmp = a[i]; a[i]=a[sel]; a[sel]=tmp; }
    }
}
