import pygame
from pygame.locals import *
from OpenGL.GL import *
from OpenGL.GLU import *
import random

# Definición de formas
def cubo(size=1):
    vertices = (
        (size,-size,-size), (size,size,-size), (-size,size,-size), (-size,-size,-size),
        (size,-size,size), (size,size,size), (-size,-size,size), (-size,size,size)
    )
    edges = (
        (0,1),(0,3),(0,4),(2,1),(2,3),(2,7),(6,3),(6,4),(6,7),(5,1),(5,4),(5,7)
    )
    glBegin(GL_LINES)
    for edge in edges:
        for vertex in edge:
            glVertex3fv(vertices[vertex])
    glEnd()

def esfera(radius=1, slices=16, stacks=16):
    quad = gluNewQuadric()
    gluSphere(quad, radius, slices, stacks)

def cilindro(radius=0.5, height=1, slices=16, stacks=16):
    quad = gluNewQuadric()
    gluCylinder(quad, radius, radius, height, slices, stacks)

# Lista de objetos
objetos = []

# Materiales
materiales = [
    ((1,0,0), "Rojo"),
    ((0,1,0), "Verde"),
    ((0,0,1), "Azul"),
    ((1,1,0), "Amarillo"),
    ((1,0,1), "Magenta"),
    ((0,1,1), "Cian")
]

def agregar_objeto(tipo, posicion, tamaño, material):
    objetos.append({
        'tipo': tipo,
        'posicion': posicion,
        'tamaño': tamaño,
        'material': material,
        'rotacion': [0, 0, 0]
    })

def dibujar_objetos():
    for obj in objetos:
        glPushMatrix()
        glTranslatef(*obj['posicion'])
        glRotatef(obj['rotacion'][0], 1, 0, 0)
        glRotatef(obj['rotacion'][1], 0, 1, 0)
        glRotatef(obj['rotacion'][2], 0, 0, 1)
        glColor3fv(obj['material'][0])
        if obj['tipo'] == 'cubo':
            cubo(obj['tamaño'])
        elif obj['tipo'] == 'esfera':
            esfera(obj['tamaño'])
        elif obj['tipo'] == 'cilindro':
            cilindro(obj['tamaño'], obj['tamaño']*2)
        glPopMatrix()

def main():
    pygame.init()
    display = (800, 600)
    pygame.display.set_mode(display, DOUBLEBUF|OPENGL)
    gluPerspective(45, (display[0]/display[1]), 0.1, 50.0)
    glTranslatef(0.0, 0.0, -10)

    # Agregar algunos objetos iniciales
    agregar_objeto('cubo', (-2, 0, 0), 0.5, random.choice(materiales))
    agregar_objeto('esfera', (0, 0, 0), 0.5, random.choice(materiales))
    agregar_objeto('cilindro', (2, 0, 0), 0.5, random.choice(materiales))

    objeto_seleccionado = None

    while True:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                quit()
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_SPACE:
                    # Agregar un nuevo objeto aleatorio
                    tipo = random.choice(['cubo', 'esfera', 'cilindro'])
                    pos = (random.uniform(-3, 3), random.uniform(-3, 3), random.uniform(-3, 3))
                    tamaño = random.uniform(0.3, 1.0)
                    material = random.choice(materiales)
                    agregar_objeto(tipo, pos, tamaño, material)
                elif event.key == pygame.K_TAB:
                    # Cambiar objeto seleccionado
                    if objeto_seleccionado is None:
                        objeto_seleccionado = 0
                    else:
                        objeto_seleccionado = (objeto_seleccionado + 1) % len(objetos)
                elif event.key in [pygame.K_LEFT, pygame.K_RIGHT, pygame.K_UP, pygame.K_DOWN]:
                    # Mover objeto seleccionado
                    if objeto_seleccionado is not None:
                        delta = 0.1
                        if event.key == pygame.K_LEFT:
                            objetos[objeto_seleccionado]['posicion'][0] -= delta
                        elif event.key == pygame.K_RIGHT:
                            objetos[objeto_seleccionado]['posicion'][0] += delta
                        elif event.key == pygame.K_UP:
                            objetos[objeto_seleccionado]['posicion'][1] += delta
                        elif event.key == pygame.K_DOWN:
                            objetos[objeto_seleccionado]['posicion'][1] -= delta

        glRotatef(1, 3, 1, 1)
        glClear(GL_COLOR_BUFFER_BIT|GL_DEPTH_BUFFER_BIT)
        
        dibujar_objetos()
        
        pygame.display.flip()
        pygame.time.wait(10)

main()
      
