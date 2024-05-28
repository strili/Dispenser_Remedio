# Dispenser de Remedio
Este trabalho aborda o desenvolvimento de um sistema de alarme automático com dispenser para medicamentos utilizando ESP-32 com RTC.
Autores: 
- Gustavo Strilicherk Pinto RA: 20.01071-0
- Henrique Baraldi Cogo RA: 21.01811-0
- Rafael Callegaris Dias RA: 21.00531-0

# Introdução
A proposta desse projeto é o desenvolvimento de um sistema automático para liberação de remédios em 2 horários determinados via código. Como também, é a implementação de um botão que permita que o usuário despeje ou abasteça o remédio quando desejar.

# Componentes Eletrônicos e Esquema Elétrico
![image](https://github.com/strili/Dispenser_Remedio/assets/171040960/66ea5dd2-d454-44e4-ae4a-57c838ab3b15) 
![image](https://github.com/strili/Dispenser_Remedio/assets/171040960/ee55de0c-f258-4c3d-bf42-9e0afa57a8ce)
![image](https://github.com/strili/Dispenser_Remedio/assets/171040960/c8a301d7-c218-4a41-896c-6ea25b6d8800)

A tabela abaixo indica os componentes utilizados e sua nomenclatura no esquema elétrico
![image](https://github.com/strili/Dispenser_Remedio/assets/171040960/bf9f3c3d-9d10-4d52-8cf5-ddfc7da97327)

# Funcionamento do sistema
Temporizadores: O sistema utiliza temporizadores para realizar tarefas em intervalos regulares, como atualizar o RTC, tratar entradas de botões e atualizar o display.

Motores: Os motores são controlados para girar em sentidos horário e anti-horário, abrindo e fechando o dispensador de remédios. Temporizadores específicos garantem que os motores operem por um tempo adequado e façam pausas necessárias entre os ciclos.

Display: O display OLED mostra informações sobre o sistema, como o horário atual, autor, versão e informações técnicas. Ele é atualizado periodicamente.

Botões e Buzzer: Os botões permitem a navegação entre telas e a ativação manual dos motores. O buzzer fornece feedback sonoro para diferentes eventos, como acionamento de botões e motores.

A proposta deste sistema é ser um dispensador automático de medicamentos. Através do código, é possível programar horários específicos para que o compartimento do medicamento se abra. No código original, os horários definidos para essa ação são 07:30:00 e 19:30:00. As imagens abaixo mostram o funcionamento real do projeto, incluindo a placa final soldada, a estrutura física e exemplos de funcionamento conforme descrito acima.

# Software

