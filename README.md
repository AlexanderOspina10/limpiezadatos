**Limpieza de datos:**

=FILTER('Base de datos'!A1:Y17700,
  BYROW('Base de datos'!A1:Y17700,
    LAMBDA(fila,
      AND(
        COUNTBLANK(fila)=0,
        COUNTIF(fila,0)=0,
        INDEX(fila,7)<>"Amazon FBA"
      )
    )
  )
)

**REMPLAZAR:**

=ARRAYFORMULA(IF(K2:K="","",IFERROR(VLOOKUP(K2:K,'Dictionary States'!A:B,2,FALSE),K2:K)))

**FORMULAS:**

**TRAER DATOS**

=BUSCARV(A2,'Resultado limpieza de datos'!A:G,7,FALSO) 

=BUSCARV(A2,'Resultado limpieza de datos'!A:I,9,FALSO) 

**FECHAS**

=SI.ERROR(
  TEXTO(
    INDICE('Resultado limpieza de datos'!B:B,
      COINCIDIR(A2,'Resultado limpieza de datos'!A:A,0)
    ),
    "[$-en]mmmm"
  ),
  ""
)

**COMISION**

=SI.ERROR(
  INDICE('Resultado limpieza de datos'!O:O,
    COINCIDIR(A2,'Resultado limpieza de datos'!A:A,0)
  ) * 5%,
  ""
)

**SALES**

=SI.ERROR(
  SI(
    INDICE('Resultado limpieza de datos'!I:I,
      COINCIDIR(A2,'Resultado limpieza de datos'!A:A,0)
    )="Amazon ISP","Jamerson",
    SI(
      INDICE('Resultado limpieza de datos'!I:I,
        COINCIDIR(A2,'Resultado limpieza de datos'!A:A,0)
      )="BC ISP","David",
      "Mario"
    )
  ),
  ""
)

**PYTHON:**

**LIMPIAR:**

text = "nif;nombre;email;teléfono;descuento\n01234567L;Lu*is González;luisgonzalez@mail.com;656343576;12.5\n71476342J;Macar*ena Ramírez;mac*arena@mail.com;692839321;8\n63823376M;Jua*n José Martínez;juan*jo@mail.com;664888233;5.2\n98376547F;Car*men Sánchez;carmen@m*ail.com;667677855;15.7"

# 1. Limpiar el texto eliminando los asteriscos
text_limpio = text.replace('*', '')

# 2. Dividir el texto en líneas
lineas = text_limpio.split('\n')

# 3. Obtener los encabezados (primera línea)
encabezados = lineas[0].split(';')

# 4. Crear el diccionario con la información organizada
datos = {}

for linea in lineas[1:]:  # Saltar la primera línea (encabezados)
    valores = linea.split(';')
    
    # El NIF es la clave del diccionario
    nif = valores[0]
    
    # Crear un diccionario interno con los demás datos
    datos[nif] = {
        'nombre': valores[1],
        'email': valores[2],
        'teléfono': valores[3],
        'descuento': float(valores[4])
    }

# Mostrar el resultado
print(datos)

**API:**

import requests
import pandas as pd

BASE_URL = "https://pokeapi.co/api/v2/"
POKEMON_IDS = [25, 52, 124]

def get_habitat(pokemon_name):
    url = f"{BASE_URL}pokemon-species/{pokemon_name}"
    response = requests.get(url).json()
    habitat = response.get("habitat")
    return habitat["name"] if habitat else None

def get_move_effect(move_url):
    move_data = requests.get(move_url).json()
    
    effect = None
    short_effect = None

    # Buscar efectos en inglés
    for entry in move_data.get("effect_entries", []):
        if entry["language"]["name"] == "en":
            effect = entry["effect"]
            short_effect = entry["short_effect"]
            break

    # Si no tiene efecto, usar damage_class
    if not effect:
        damage_class = move_data.get("damage_class")
        effect = damage_class["name"] if damage_class else None

    return effect, short_effect

data = []

for pokemon_id in POKEMON_IDS:
    pokemon_url = f"{BASE_URL}pokemon/{pokemon_id}"
    pokemon_response = requests.get(pokemon_url).json()

    pokemon_name = pokemon_response["name"]
    habitat = get_habitat(pokemon_name)

    for move in pokemon_response["moves"]:
        move_name = move["move"]["name"]
        move_url = move["move"]["url"]

        effect, short_effect = get_move_effect(move_url)

        data.append({
            "id": pokemon_id,
            "pokemon": pokemon_name,
            "movimiento": move_name,
            "efectos": effect,
            "efecto_corto": short_effect,
            "habitat": habitat
        })

df = pd.DataFrame(data)
df.head()


