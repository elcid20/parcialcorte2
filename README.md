#include <stdio.h>
#include <stdlib.h>
#include <string.h>

struct pelicula {
    char nombre[50];
    int anio;
    char genero[30];
    float recaudacion;
    struct pelicula *izq;
    struct pelicula *der;
};

struct pelicula *raiz = NULL, *aux;

int insertar(struct pelicula *nodo, struct pelicula *nuevo) {
    if (nuevo->anio < nodo->anio) {
        if (nodo->izq == NULL) {
            nodo->izq = nuevo;
            return 1;
        } else {
            return insertar(nodo->izq, nuevo);
        }
    } else if (nuevo->anio > nodo->anio) {
        if (nodo->der == NULL) {
            nodo->der = nuevo;
            return 1;
        } else {
            return insertar(nodo->der, nuevo);
        }
    } else {
        printf("Error: Ya existe una película con el año %d. No se puede insertar.\n", nuevo->anio);
        free(nuevo);
        return 0;
    }
}

void registrarPelicula() {
    aux = (struct pelicula *) malloc(sizeof(struct pelicula));
    aux->izq = aux->der = NULL;
    printf("Nombre de la pelicula: ");
    getchar(); 
    fgets(aux->nombre, 50, stdin);
    aux->nombre[strcspn(aux->nombre, "\n")] = '\0'; 
    printf("Anio de realizacion: ");
    scanf("%d", &aux->anio);
    printf("Genero: ");
    getchar();
    fgets(aux->genero, 30, stdin);
    aux->genero[strcspn(aux->genero, "\n")] = '\0';
    printf("Recaudacion (en millones): ");
    scanf("%f", &aux->recaudacion);

    if (raiz == NULL) {
        raiz = aux;
    } else {
        insertar(raiz, aux);
    }
}

void inorden(struct pelicula *nodo) {
    if (nodo != NULL) {
        inorden(nodo->izq);
        printf("Nombre: %s | Año: %d | Genero: %s | Recaudacion: %.2fM\n",
               nodo->nombre, nodo->anio, nodo->genero, nodo->recaudacion);
        inorden(nodo->der);
    }
}

void preorden(struct pelicula *nodo) {
    if (nodo != NULL) {
        printf("Nombre: %s | Año: %d | Genero: %s | Recaudacion: %.2fM\n",
               nodo->nombre, nodo->anio, nodo->genero, nodo->recaudacion);
        preorden(nodo->izq);
        preorden(nodo->der);
    }
}

void posorden(struct pelicula *nodo) {
    if (nodo != NULL) {
        posorden(nodo->izq);
        posorden(nodo->der);
        printf("Nombre: %s | Año: %d | Genero: %s | Recaudacion: %.2fM\n",
               nodo->nombre, nodo->anio, nodo->genero, nodo->recaudacion);
    }
}

void buscarPorNombre(struct pelicula *nodo, char *nombre) {
    if (nodo != NULL) {
        buscarPorNombre(nodo->izq, nombre);
        if (strcmp(nodo->nombre, nombre) == 0) {
            printf("Pelicula encontrada: %s | Año: %d | Genero: %s | Recaudacion: %.2fM\n",
                   nodo->nombre, nodo->anio, nodo->genero, nodo->recaudacion);
        }
        buscarPorNombre(nodo->der, nombre);
    }
}

void mostrarPorGenero(struct pelicula *nodo, char *genero) {
    if (nodo != NULL) {
        mostrarPorGenero(nodo->izq, genero);
        if (strcmp(nodo->genero, genero) == 0) {
            printf("Nombre: %s | Año: %d | Recaudacion: %.2fM\n",
                   nodo->nombre, nodo->anio, nodo->recaudacion);
        }
        mostrarPorGenero(nodo->der, genero);
    }
}

void fracasosTaquilleros(struct pelicula *nodo, struct pelicula **lista, int *n) {
    if (nodo != NULL) {
        fracasosTaquilleros(nodo->izq, lista, n);
        lista[*n] = nodo;
        (*n)++;
        fracasosTaquilleros(nodo->der, lista, n);
    }
}

int comparar(const void *a, const void *b) {
    struct pelicula *p1 = *(struct pelicula **)a;
    struct pelicula *p2 = *(struct pelicula **)b;

    if (p1->recaudacion > p2->recaudacion) {
        return 1;
    } else if (p1->recaudacion < p2->recaudacion) {
        return -1;
    } else {
        return 0;
    }
}

struct pelicula* encontrarMinimo(struct pelicula* nodo) {
    while (nodo->izq != NULL) {
        nodo = nodo->izq;
    }
    return nodo;
}

struct pelicula* eliminarPorNombre(struct pelicula* nodo, char* nombre, int* eliminado) {
    if (nodo == NULL) return NULL;

    if (strcmp(nombre, nodo->nombre) < 0) {
        nodo->izq = eliminarPorNombre(nodo->izq, nombre, eliminado);
    } else if (strcmp(nombre, nodo->nombre) > 0) {
        nodo->der = eliminarPorNombre(nodo->der, nombre, eliminado);
    } else {
        *eliminado = 1;
        if (nodo->izq == NULL && nodo->der == NULL) {
            free(nodo);
            return NULL;
        } else if (nodo->izq == NULL) {
            struct pelicula* temp = nodo->der;
            free(nodo);
            return temp;
        } else if (nodo->der == NULL) {
            struct pelicula* temp = nodo->izq;
            free(nodo);
            return temp;
        } else {
            struct pelicula* sucesor = encontrarMinimo(nodo->der);
            strcpy(nodo->nombre, sucesor->nombre);
            nodo->anio = sucesor->anio;
            strcpy(nodo->genero, sucesor->genero);
            nodo->recaudacion = sucesor->recaudacion;
            nodo->der = eliminarPorNombre(nodo->der, sucesor->nombre, eliminado);
        }
    }
    return nodo;
}

int main() {
    int opcion;
    char nombreBuscado[50], generoBuscado[30];
    struct pelicula *lista[100];
    int total = 0;

    do {
        printf("\n1. Registrar Pelicula\n");
        printf("2. Mostrar Inorden\n");
        printf("3. Mostrar Preorden\n");
        printf("4. Mostrar Posorden\n");
        printf("5. Buscar por Nombre\n");
        printf("6. Mostrar por Genero\n");
        printf("7. Mostrar 3 Fracasos Taquilleros\n");
        printf("8. Eliminar por Nombre\n");
        printf("9. Salir\n");
        printf("Seleccione una opcion: ");
        scanf("%d", &opcion);

        switch (opcion) {
            case 1: registrarPelicula(); break;
            case 2: inorden(raiz); break;
            case 3: preorden(raiz); break;
            case 4: posorden(raiz); break;
            case 5:
                printf("Nombre a buscar: ");
                getchar();
                fgets(nombreBuscado, 50, stdin);
                nombreBuscado[strcspn(nombreBuscado, "\n")] = '\0';
                buscarPorNombre(raiz, nombreBuscado);
                break;
            case 6:
                printf("Género a buscar: ");
                getchar();
                fgets(generoBuscado, 30, stdin);
                generoBuscado[strcspn(generoBuscado, "\n")] = '\0';
                mostrarPorGenero(raiz, generoBuscado);
                break;
            case 7:
                total = 0;
                fracasosTaquilleros(raiz, lista, &total);
                qsort(lista, total, sizeof(struct pelicula *), comparar);
                printf("3 Fracasos taquilleros:\n");
                for (int i = 0; i < total && i < 3; i++) {
                    printf("Nombre: %s | Recaudacion: %.2fM\n",
                           lista[i]->nombre, lista[i]->recaudacion);
                }
                break;
            case 8:
                printf("Nombre de la pelicula a eliminar: ");
                getchar();
                fgets(nombreBuscado, 50, stdin);
                nombreBuscado[strcspn(nombreBuscado, "\n")] = '\0';
                {
                    int eliminado = 0;
                    raiz = eliminarPorNombre(raiz, nombreBuscado, &eliminado);
                    if (eliminado)
                        printf("Pelicula eliminada exitosamente.\n");
                    else
                        printf("Pelicula no encontrada.\n");
                }
                break;
        }
    } while (opcion != 9);

    return 0;
}
