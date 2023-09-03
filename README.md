# Proyecto-de-Embebidos
# George Maquilon - Ricardo Romero

# Descripción

En la actualidad, la población adulta mayor enfrenta diversos desafíos relacionados con el uso de medicamentos, ya que suelen consumir una cantidad considerable de ellos y son más susceptibles a la falta de una adecuada supervisión. En este contexto, los enfermeros desempeñan un papel fundamental al documentar las indicaciones de nuevos medicamentos, mantener actualizado un registro de los fármacos que el paciente consume y realizar revisiones periódicas para asegurarse de que se disponga de suficiente stock y evaluar la necesidad de los medicamentos recetados. Sin embargo, es necesario mejorar y fortalecer estas prácticas para garantizar una atención óptima y segura para los adultos mayores. El dispensador de pastillas programado incorporará un LCD que mostrará el horario específico en el que cada adulto mayor debe tomar sus medicamentos, lo que garantizará un mejor cumplimiento del tratamiento. Únicamente el personal de enfermería tendrá la capacidad de modificar los horarios de administración a través del código fuente que proporcionará opciones para cada espacio de pastillas. Además, se utilizará tres diodos leds para diferenciar las distintas pastillas. Estos componentes emitirán una luz de color único, permitiendo una identificación clara y precisa de cada medicamento.

![image](https://github.com/goergito/Proyecto-de-Embebidos/assets/137368787/d5c87343-4b17-4b51-af02-b39615f9efc8)


Para garantizar la seguridad del usuario asignado al dispensador, se implementará un sistema de control de acceso con dos servomotores. Esta medida garantizará la distribución y administración de los medicamentos de manera adecuada ya que el primer servomotor se encargará de mover el recipiente dividido donde se encontrarán los tres tipos de pastillas (0° -> Paracetamol, 90° -> Ibuprofeno, 180° -> LemonFlu) y el segundo servomotor se encargará de la apertura de una compuerta (0° a 90°) para poder tomar las pastillas.
El sistema intentará conectarse a una red Wifi previamente configurada mediante la ESP32. Esto permitirá que el usuario final pueda conectarse a través de la zona horario de Ecuador (Zona horario de Quito – GMT-5) vinculando la hora local y pudiendo así programar los horarios deseados para la toma de cada tipo de pastillas.
