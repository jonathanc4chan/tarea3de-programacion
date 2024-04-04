codigo del programa de Arboles B

package tarea3progra;

import java.util.ArrayList;

import java.util.HashSet;

import java.util.List;

import java.util.Scanner;

import java.util.Set;

class Nodo {
    List<Integer> claves;
    List<Nodo> hijos;
    boolean esHoja;

    Nodo(int grado, boolean esHoja) {
        this.claves = new ArrayList<>(grado - 1);
        this.hijos = new ArrayList<>(grado);
        this.esHoja = esHoja;
    }
}

public class ArbolB {
    private int grado;

    ArbolB(int grado) {
        this.grado = Math.max(2, grado); // Garantiza un grado mínimo de 2
    }

    public Nodo insertar(Nodo raiz, int clave) {
        if (raiz == null) {
            raiz = new Nodo(grado, true);
            raiz.claves.add(clave);
            return raiz;
        }

        if (raiz.claves.size() < grado - 1) {
            insertarEnNodoNoLleno(raiz, clave);
        } else {
            Nodo nuevaRaiz = new Nodo(grado, false);
            nuevaRaiz.hijos.add(raiz);
            splitNodo(nuevaRaiz, 0, raiz);
            insertarEnNodoNoLleno(nuevaRaiz, clave);
            raiz = nuevaRaiz;
        }
        return raiz;
    }

    private void insertarEnNodoNoLleno(Nodo nodo, int clave) {
        int i = nodo.claves.size() - 1;

        if (nodo.esHoja) {
            nodo.claves.add(clave);
            nodo.claves.sort(null);
        } else {
            while (i >= 0 && clave < nodo.claves.get(i)) {
                i--;
            }

            i++;
            Nodo hijo = nodo.hijos.get(i);
            if (hijo.claves.size() == grado - 1) {
                splitNodo(nodo, i, hijo);
                if (clave > nodo.claves.get(i)) {
                    i++;
                }
            }
            insertarEnNodoNoLleno(nodo.hijos.get(i), clave);
        }
    }

    private void splitNodo(Nodo padre, int indice, Nodo hijo) {
        Nodo nuevoHijo = new Nodo(grado, hijo.esHoja);
        nuevoHijo.claves.addAll(hijo.claves.subList(grado / 2, grado - 1));
        hijo.claves.subList(grado / 2, grado - 1).clear();

        if (!hijo.esHoja) {
            nuevoHijo.hijos.addAll(hijo.hijos.subList(grado / 2, grado));
            hijo.hijos.subList(grado / 2, grado).clear();
        }

        padre.claves.add(indice, hijo.claves.get(grado / 2 - 1));
        padre.hijos.add(indice + 1, nuevoHijo);
    }

    public boolean buscar(Nodo raiz, int clave) {
        if (raiz == null) {
            return false;
        }
        int i = 0;
        while (i < raiz.claves.size() && clave > raiz.claves.get(i)) {
            i++;
        }
        if (i < raiz.claves.size() && clave == raiz.claves.get(i)) {
            return true;
        } else if (raiz.esHoja) {
            return false;
        } else {
            return buscar(raiz.hijos.get(i), clave);
        }
    }

    public Nodo eliminar(Nodo raiz, int clave) {
        if (raiz == null) {
            System.out.println("El árbol está vacío.");
            return null;
        }

        boolean eliminado = eliminarRecursivo(raiz, clave);
        if (!eliminado) {
            System.out.println("La clave " + clave + " no existe en el árbol.");
        }
        return raiz;
    }

    private boolean eliminarRecursivo(Nodo nodo, int clave) {
        int indice = nodo.claves.indexOf(clave);

        if (indice != -1) { // Clave encontrada en este nodo
            if (nodo.esHoja) {
                nodo.claves.remove(indice);
                System.out.println("La clave " + clave + " ha sido eliminada.");
            } else {
                Nodo predecesor = nodo.hijos.get(indice);
                while (!predecesor.esHoja) {
                    predecesor = predecesor.hijos.get(predecesor.hijos.size() - 1);
                }
                int predecesorClave = predecesor.claves.get(predecesor.claves.size() - 1);
                nodo.claves.set(indice, predecesorClave);
                eliminarRecursivo(nodo.hijos.get(indice), predecesorClave);
            }
            return true;
        } else { // Clave no encontrada en este nodo
            int i = 0;
            while (i < nodo.claves.size() && clave > nodo.claves.get(i)) {
                i++;
            }
            if (i < nodo.claves.size()) {
                Nodo hijo = nodo.hijos.get(i);
                if (hijo.claves.size() >= grado / 2) {
                    return fusionarHijosSiEsNecesario(nodo, i);
                } else {
                    if (i != nodo.claves.size() && nodo.hijos.get(i + 1).claves.size() >= grado / 2) {
                        return redistribuirHijosSiEsNecesario(nodo, i);
                    } else if (i != 0 && nodo.hijos.get(i - 1).claves.size() >= grado / 2) {
                        return redistribuirHijosSiEsNecesario(nodo, i - 1);
                    } else {
                        if (i != nodo.claves.size()) {
                            return fusionarHijosSiEsNecesario(nodo, i);
                        } else {
                            return fusionarHijosSiEsNecesario(nodo, i - 1);
                        }
                    }
                }
            } else {
                System.out.println("Error: Índice fuera de límites al eliminar.");
                return false;
            }
        }
    }

    private boolean fusionarHijosSiEsNecesario(Nodo nodo, int indice) {
        if (indice >= 0 && indice < nodo.hijos.size() - 1) {
            Nodo hijo = nodo.hijos.get(indice);
            Nodo hermano = nodo.hijos.get(indice + 1);

            if (hijo.claves.size() < hermano.claves.size()) {
                hijo.claves.add(nodo.claves.get(indice));
                nodo.claves.set(indice, hermano.claves.remove(0));
                if (!hijo.esHoja) {
                    hijo.hijos.add(hermano.hijos.remove(0));
                }
            } else {
                hermano.claves.add(0, nodo.claves.get(indice));
                nodo.claves.set(indice, hijo.claves.remove(hijo.claves.size() - 1));
                if (!hermano.esHoja) {
                    hermano.hijos.add(0, hijo.hijos.remove(hijo.hijos.size() - 1));
                }
            }

            return eliminarRecursivo(nodo.hijos.get(indice), nodo.claves.get(indice));
        } else {
            System.out.println("Error: Índice fuera de límites al fusionar hijos.");
            return false;
        }
    }

    private boolean redistribuirHijosSiEsNecesario(Nodo nodo, int indice) {
        if (indice >= 0 && indice < nodo.hijos.size() - 1) {
            Nodo hijo = nodo.hijos.get(indice);
            Nodo hermano = nodo.hijos.get(indice + 1);

            if (hijo.claves.size() < hermano.claves.size()) {
                hijo.claves.add(nodo.claves.get(indice));
                nodo.claves.set(indice, hermano.claves.remove(0));
                if (!hijo.esHoja) {
                    hijo.hijos.add(hermano.hijos.remove(0));
                }
            } else {
                hermano.claves.add(0, nodo.claves.get(indice));
                nodo.claves.set(indice, hijo.claves.remove(hijo.claves.size() - 1));
                if (!hermano.esHoja) {
                    hermano.hijos.add(0, hijo.hijos.remove(hijo.hijos.size() - 1));
                }
            }

            return eliminarRecursivo(nodo.hijos.get(indice), nodo.claves.get(indice));
        } else {
            System.out.println("Error: Índice fuera de límites al redistribuir hijos.");
            return false;
        }
    }

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        System.out.println("Bienvenido al programa de Árbol B");
        System.out.println("Un Árbol B es una estructura de datos de búsqueda que mantiene claves ordenadas y permite búsquedas, inserciones y eliminaciones eficientes.");
        System.out.println("Ingrese el grado del árbol B (debe ser un número entero mayor o igual a 2):");
        int grado = scanner.nextInt();

        ArbolB arbolB = new ArbolB(grado);
        Nodo raiz = null;
        boolean continuar = true;

        while (continuar) {
            System.out.println("\nSeleccione una opción:");
            System.out.println("1. Insertar claves");
            System.out.println("2. Eliminar clave");
            System.out.println("3. Buscar clave");
            System.out.println("4. Salir");
            System.out.print("Opción seleccionada: ");
            int opcion = scanner.nextInt();

            switch (opcion) {
                case 1:
                    System.out.println("Ingrese la cantidad de claves a insertar:");
                    int numClaves = scanner.nextInt();
                    Set<Integer> clavesInsertadas = new HashSet<>();
                    System.out.println("Ingrese las claves:");
                    for (int i = 0; i < numClaves; i++) {
                        int claveInsertar = scanner.nextInt();
                        if (clavesInsertadas.contains(claveInsertar)) {
                            System.out.println("La clave " + claveInsertar + " ya ha sido ingresada. Intente con otra clave.");
                            i--;
                        } else {
                            clavesInsertadas.add(claveInsertar);
                        }
                    }
                    for (int clave : clavesInsertadas) {
                        raiz = arbolB.insertar(raiz, clave);
                        System.out.println("Clave " + clave + " insertada correctamente.");
                    }
                    break;
                case 2:
                    System.out.println("Ingrese la clave a eliminar:");
                    int claveEliminar = scanner.nextInt();
                    raiz = arbolB.eliminar(raiz, claveEliminar);
                    break;
                case 3:
                    System.out.println("Ingrese la clave a buscar:");
                    int claveBuscar = scanner.nextInt();
                    boolean encontrado = arbolB.buscar(raiz, claveBuscar);
                    if (encontrado) {
                        System.out.println("La clave " + claveBuscar + " está presente en el árbol.");
                    } else {
                        System.out.println("La clave " + claveBuscar + " no está presente en el árbol.");
                    }
                    break;
                case 4:
                    continuar = false;
                    break;
                default:
                    System.out.println("Opción no válida");
            }
        }
    }
}
