# Videojuego-Zombijungle
Aqui se agregaran los scripts utilizados para que el personaje se mueva y funcione el videojuego y su documentacion.

extends CharacterBody2D  # Para 2D

# Variables
@export var velocidad = 200.0  # Velocidad de movimiento
@export var gravedad = 830.0   # Valor de la gravedad
@export var salto_fuerza = -430.0  # Fuerza del salto (negativa porque es hacia arriba)
@export var tiempo_ataque = 0.5  # Duración del ataque en segundos
var direccion = Vector2.ZERO   # Dirección de movimiento
var animacion  # Variable para almacenar la referencia a AnimatedSprite2D
var puede_atacar = true  # Control de si el personaje puede atacar
var esta_atacando = false  # Verifica si el personaje está en animación de ataque
var temporizador_ataque = 0.0  # Controla el tiempo del ataque

# Esta función se llama cuando la escena está lista
func _ready():
	animacion = $AnimatedSprite2D  # Aquí obtenemos la referencia al nodo AnimatedSprite2D

func _physics_process(delta):
	direccion.x = 0  # Reiniciar la dirección horizontal cada frame

	# Verificar si el personaje está atacando
	if esta_atacando:
		temporizador_ataque -= delta
		if temporizador_ataque <= 0:
			esta_atacando = false  # El ataque ha terminado
			puede_atacar = true  # Permitir otro ataque

	# Controles de movimiento (izquierda y derecha) solo si no está atacando
	if not esta_atacando:
		if Input.is_action_pressed("ui_left"):
			direccion.x -= 1
		if Input.is_action_pressed("ui_right"):
			direccion.x += 1

	# Normalizar la dirección para evitar que sea más rápido en diagonal
	if direccion != Vector2.ZERO:
		direccion = direccion.normalized()

	# Mover horizontalmente al personaje
	velocity.x = direccion.x * velocidad

	# Aplicar la gravedad en el eje Y (vertical)
	if not is_on_floor():  # Solo aplica gravedad si no está en el suelo
		velocity.y += gravedad * delta

	# Control de salto: solo permitir el salto si el personaje está tocando el suelo y no está atacando
	if Input.is_action_just_pressed("ui_accept") and is_on_floor() and not esta_atacando:
		velocity.y = salto_fuerza  # Aplicar fuerza de salto

	# Verificar ataque (usando la tecla Ctrl, que has asignado en el Input Map)
	if Input.is_action_just_pressed("ui_attack") and puede_atacar:
		ataque()  # Ejecutar ataque

	# Mover al personaje usando move_and_slide con la dirección del suelo
	move_and_slide()

	# Controlar la animación
	if esta_atacando:
		animacion.play("attack")  # Reproducir la animación de ataque
	elif not is_on_floor():
		animacion.play("jump")  # Si el personaje está en el aire, reproducir la animación de salto
	elif direccion.x != 0:
		# Si hay movimiento horizontal
		if velocity.x > 0:
			animacion.play("run")  # Reproducir la animación de correr
			animacion.flip_h = false  # Asegurarse de que no esté volteado a la izquierda
		else:
			animacion.play("run")
			animacion.flip_h = true  # Voltear a la izquierda
	elif is_on_floor():
		animacion.play("idle")  # Si no se mueve y está en el suelo, reproducir la animación de idle

# Función de ataque
func ataque():
	esta_atacando = true  # Activar el estado de ataque
	puede_atacar = false  # No permitir más ataques hasta que termine
	temporizador_ataque = tiempo_ataque  # Iniciar temporizador del ataque
	# Aquí podrías agregar lógica para dañar a enemigos o efectos adicionales.
