# Semáforos - Semaphores
- Contador de recursos compartilhados
	- Tipo de Variável
- Recurso compartilhado
	- Mesas em um restaurante 
	- 10 mesas livres -> semáforo armazena valor 10
	- 1 Banheiro livre -> semáforo armazena valor 1
- Operação down
	- pedido para usar recurso
	- "protege" o que tem embaixo
	- Semáforo é decrementado
	- Se for zero, thread é bloqueada
		- Espera suspensa (idle wait) em uma queue
			- idle wait é menos custosa que busy wait
- Operação up
	- Libera recurso

# Semáforo em <semaphore.h>
Global
```C
#include <semaphore.h>
sem_t semaphore;
```

Main
```C
int shared = 0;
unsigned int value = 1;
sem_init(&semaphore, shared, value);
```

Região Crítica
```C
sem_wait(&semaphore);
balance++;
sem_post(&semaphore);
```

# Semáforo em <pthread.h>
- mutex significa mutual exclusion
- só aceita 0 ou 1

Global
```C
pthread_mutex_t mutex_semaphore;
```

Região Crítica
```C
pthread_mutex_lock(&mutex_semaphore);
balance++;
pthread_mutex_unlock(&mutex_semaphore);
```

# Modelo Produtor-Consumidor
- Controlar acesso de recurso que é escrito e utilizado
- Exemplo: produção e uso de imagem
	- Câmera gravando + identificação de imagem (rostos, armas, OCR, etc)

- Dado um buffer que é escrito e utilizado
	- Produtor preenche buffer
	- Consumidor remove do buffer

- 3 Semáforos
	- Semáforo para controlar escritas no buffer
	- Semáforo para controlar consumo no buffer
	- Semáforo para controlar acesso ao buffer

```
lock_producer -> lock_buffer -> produce -> unlock_buffer -> unlock_consumer

lock_consumer -> lock_buffer -> consume -> unlock_buffer -> unlock_producer
```

``` C
void *produce(void *x) {
    int producer_number = *(int *)x;
		
    for (int i = 0; i < 10; i++) {
        int product = rand();
		
        sem_wait(&producer_semaphore);
        pthread_mutex_lock(&buffer_semaphore);
		
        buffer[buffer_end] = product;
        printf("%d\tproduzido\tpelo\tprodutor\t%d\te armazenado\n", buffer[buffer_end], producer_number);
        buffer_end = (buffer_end + 1) % N;
		
        pthread_mutex_unlock(&buffer_semaphore);
        sem_post(&consumer_semaphore);
    }
	
    return NULL;
}
```

``` C
void *consume(void *x) {
    int consumer_number = *(int *)x;
		
    for (int i = 0; i < 10; i++) {
        sem_wait(&consumer_semaphore);
        pthread_mutex_lock(&buffer_semaphore);
		
        printf("%d\tconsumido\tpelo\tconsumidor\t%d\n", buffer[buffer_start], consumer_number);
        buffer_start = (buffer_start + 1) % N;
		
        pthread_mutex_unlock(&buffer_semaphore);
        sem_post(&producer_semaphore);
    }
	
    return NULL;
}
```

- Analogia loucona
	- Restaurante com 1 fila de cozinheiros, 1 fila de consumidores e fogão com 10 bocas
	- Cozinheiro só pode cozinhar quando fogão ta livre e alguma boca ta livre
	- Consumidor só pode comer quando fogão ta livre e alguma boca tem comida 

#Modelo Jantar dos Filósofos
