# Dispenser de Remedio
Este trabalho aborda o desenvolvimento de um sistema de alarme automático com dispenser para medicamentos utilizando ESP-32 com RTC.
Autores: 
- Gustavo Strilicherk Pinto RA: 20.01071-0
- Henrique Baraldi Cogo RA: 21.01811-0
- Rafael Callegaris Dias RA: 21.00531-0
- Lucas Silva Anholeto RA: 21.02145-7

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

    #include "shared_data.h" //biblioteca criada...

    String deviceName   = "Dispenser";    //nome do dispositivo
    String autorName = "Gustavo";
    String Version      = "0.0";
    String dataVersion  = "01/05/2024 18:05";

    //criacao das flags para timers
    char fs_1ms   = 0;
    char fs_10ms   = 0;
    char fs_100ms = 0;   //flags de estado para cada x segundos
    char fs_60s   = 0;
    char fs_1s    = 0;

    unsigned long millis_atual_1ms;
    unsigned long millis_atual_10ms;
    unsigned long millis_atual_100ms;
    unsigned long millis_atual_1s;
    unsigned long millis_atual_60s;

    Buzzer buzzer;
    Io io;
    Lcd lcd;
    Led led;
    Motor motor;
    Relogio relogio;
    Tela tela;

    RTC_DS3231 rtc; //RTC da classe DS3132 (própria biblioteca do módulo)

    void setup()
    {
    Serial.begin(115200); //monitor serial

    inicializa_io()   < 0 ?  Serial.println("erro em inicializa_io") :  Serial.println("inicializa_io ok");
    testa_display()   < 0 ?  Serial.println("erro no display"      ) :  Serial.println("display ok"      );
    inicializa_rtc()  < 0 ?  Serial.println("erro no rtc"          ) :  Serial.println("rtc ok"          );

    Serial.println("Starting " +  deviceName );
    inicializa_sistema();
    }
  
    void loop()
    {
    trata_timers();

    if (fs_1ms)
    {
    fs_1ms = 0;
    atualiza_rtc();
    trata_io();
    trata_horario();
    trata_motor();
    }
    if (fs_10ms)
    {
    fs_10ms = 0;
    }
    if (fs_100ms)
    {
    fs_100ms = 0;
    atualiza_display();
    }

    if (fs_60s)
    {
    fs_60s = 0;

    }
    }

    char inicializa_io (void)
    {
    char ret;

    ret = -1;
    pinMode(BUTTON_MENU_PIN,INPUT);
    pinMode(BUTTON_M1_PIN,  INPUT);
    pinMode(BUTTON_M2_PIN,  INPUT);
    pinMode(MOTOR_1A,      OUTPUT);
    pinMode(MOTOR_1B,      OUTPUT);
    pinMode(MOTOR_2A,      OUTPUT);
    pinMode(MOTOR_2B,      OUTPUT);
    pinMode(BUZZER_PIN,    OUTPUT);

    digitalWrite(BUZZER_PIN, LOW);
    digitalWrite(MOTOR_1A,   LOW);
    digitalWrite(MOTOR_1B,   LOW);
    digitalWrite(MOTOR_2A,   LOW);
    digitalWrite(MOTOR_2B,   LOW);

    ret = 0;

    return ret; //inicializa_io = valor ret
    }

    void trata_io(void)
    {
    inicializa_buzzer();
    trata_buzzer();
    debounce_botao(); //debounce de 100ms
    trata_botao();
    }

    void trata_timers(void)
    {
     /*      Descrição de funcionamento:

         Definição de tempo de timers
    */

    unsigned long t_atual;
    t_atual = millis();  // função milis retorna o tempo que o microcontrolador está ligado
    if (t_atual - millis_atual_1ms >= 1) //1ms
    {
    millis_atual_1ms = t_atual;
    fs_1ms = 1;
    }

    if (t_atual - millis_atual_10ms >= 9) //10ms
    {
    millis_atual_10ms = t_atual;
    fs_10ms = 1;
    }

    if (t_atual - millis_atual_100ms >= 99) //100ms
    {
    millis_atual_100ms = t_atual;
    fs_100ms = 1;
    }
    if (t_atual - millis_atual_1s >= 999) //1s
    {
      millis_atual_1s = t_atual;
    fs_1s = 1;
    }
    if (t_atual - millis_atual_60s >= 59999) //60s
    {
    millis_atual_60s = t_atual;
    fs_60s = 1;
    }
    }

    char testa_display(void)
    {
    char ret;
    ret = -1;
    display.begin(SSD1306_SWITCHCAPVCC, 0x3C);
    ret = 0;
    return ret;
    }
    char inicializa_rtc(void)
    {
    char ret;
    ret = -1;
    rtc.begin();
    //rtc.adjust(DateTime(F(__DATE__), F(__TIME__)));  // Atualiza o horário automaticamente, necessário na primeira compilação e depois comentar
    ret = 0;
    return ret;
    }

    void inicializa_sistema(void)
    {
    lcd.tela = INICIO;         //inicializa o sistema sempre na tela de Início
    io.buzzer_inicializa = 1;  //inicialização do sistema + buzzer
    }

    void inicializa_buzzer(void)
    {
    if (io.buzzer_inicializa)
    {
    io.buzzer_inicializa = 0;    
    io.buzzer_on         = 1;     //acionar buzzer
    io.buzzer_duplo_on   = 1;     //determinar acionar beep
    io.buzzer_tipo = BEEP_DUPLO;  //tipo de beep: duplo
    }
    }

    void trata_buzzer(void)
      {
    static unsigned int contador_buzzer_simples  = 0; //static mantém valor da variável
    static unsigned int contador_buzzer_duplo    = 0; //static mantém valor da variável
    static unsigned int contador_duplo_aux_low   = 0; //static mantém valor da variável
    static unsigned int contador_duplo_aux_high  = 0; //static mantém valor da variável

    if (io.buzzer_on)
    {
    switch (io.buzzer_tipo)
    {

      case BEEP_SIMPLES:

        digitalWrite(BUZZER_PIN, HIGH);
        io.buzzer_simples_on = 1;
        if (io.buzzer_simples_on)
        {
          contador_buzzer_simples++;
          if (contador_buzzer_simples > TEMPO_SIMPLES)
          {
            contador_buzzer_simples = 0;
            digitalWrite(BUZZER_PIN, LOW);
            io.buzzer_on = 0;
          }
        }
        else
        {
          contador_buzzer_simples = 0;
          digitalWrite(BUZZER_PIN, LOW);
        }
        break;

      case BEEP_DUPLO:
      
        if (io.buzzer_duplo_on)
        {
          digitalWrite(BUZZER_PIN, HIGH);
          contador_buzzer_duplo++;                        //tempo para manter primeiro beep em nível alto
          if (contador_buzzer_duplo > TEMPO_DUPLO_CURTO)
          {
            contador_buzzer_duplo = 0;
            io.buzzer_duplo_on = 0;
            io.buzzer_tempo_duplo_low = 1;
            digitalWrite(BUZZER_PIN, LOW);
          }
        }

        if (io.buzzer_duplo_on == 0 && io.buzzer_tempo_duplo_low)
        {
          contador_duplo_aux_low++;                      //tempo para manter mudo entre primeiro e segundo beep
          if (contador_duplo_aux_low > TEMPO_DUPLO_AUX_LOW)
          {
            contador_duplo_aux_low = 0;
            io.buzzer_duplo_on    = 0;
            io.buzzer_tempo_duplo_low  = 0;
            io.buzzer_tempo_duplo_high = 1;
            digitalWrite(BUZZER_PIN, HIGH);
          }
        }

          if (io.buzzer_duplo_on == 0 && io.buzzer_tempo_duplo_high)
          {
            contador_duplo_aux_high++;                    //tempo para manter segundo beep em nível alto
            if (contador_duplo_aux_high > TEMPO_DUPLO_AUX_HIGH)
            {
              io.buzzer_tempo_duplo_high = 0;
              io.buzzer_on = 0;
              contador_duplo_aux_high = 0;
              digitalWrite(BUZZER_PIN, LOW);
            }
          }
          break;
      
      case BEEP_MOTOR:
        static unsigned int contador_beep_motor = 0;
        contador_beep_motor++;
        if (contador_beep_motor > TEMPO_BEEP_MOTOR)
        {
          contador_beep_motor = 0;
          io.buzzer_motor = !io.buzzer_motor;
        }
        (io.buzzer_motor) ? digitalWrite(BUZZER_PIN, HIGH) : digitalWrite(BUZZER_PIN, LOW);
          break;
        }
    }
    else
    {
      digitalWrite(BUZZER_PIN, LOW);
    }
    }

    void atualiza_rtc(void)
    {
    DateTime agora = rtc.now();

    relogio.segundos = agora.second();  //definição do segundo atual
    relogio.minutos  = agora.minute();  //definição do minuto atual
    relogio.hora     = agora.hour() ;   //definição da hora atual

    sprintf(relogio.formata_msg, "%02d:%02d:%02d", relogio.hora, relogio.minutos, relogio.segundos); // montando string com char
    }

    void debounce_botao (void)
    {
    static unsigned int contador_botao_menu = 0;   //static mantém valor da variável
    static unsigned int contador_botao_m1   = 0;   //static mantém valor da variável
    static unsigned int contador_botao_m2   = 0;   //static mantém valor da variável

    (digitalRead(BUTTON_MENU_PIN)) == LOW ? io.botao_menu_on = 1 : io.botao_menu_on = 0;
    (digitalRead(BUTTON_M1_PIN))   == LOW ? io.botao_m1_on   = 1 : io.botao_m1_on   = 0;
    (digitalRead(BUTTON_M2_PIN))   == LOW ? io.botao_m2_on   = 1 : io.botao_m2_on   = 0;

    if (io.botao_menu_on)
    {
      contador_botao_menu++;
      if (contador_botao_menu > TEMPO_DEBOUCING_MENU)
      {
        contador_botao_menu  = 0;
        io.botao_menu_on     = 0;
        io.botao_menu_aciona = 1;       //flag para confirmar botao acionado
        io.buzzer_on         = 1;       //para ativar buzzer
        io.buzzer_tipo = BEEP_SIMPLES;  //para definir só um beep
      }
      else
      {
        io.botao_menu_aciona = 0;
      }
    }

    if (io.botao_m1_on && relogio.aciona_horario == 0) //AND com relogio para garantir que quando o relogio esteja ativado, não seja possível acionar botao
    {
      contador_botao_m1++;
      if(contador_botao_m1 > TEMPO_DEBOUCING_MOTOR)
      {
        contador_botao_m1  = 0;
        io.botao_m1_on     = 0;
        io.buzzer_on       = 1;         //para ativar buzzer
        io.buzzer_tipo = BEEP_SIMPLES;  //para definir o beep do motor
        io.motor_on        = 1;         //flag de motor ON 
        io.numero_motor    = MOTOR_1;  //para definir botao 1 = motor 1
        io.aciona_motor_1_horario = 1; //para ligar motor 1
      }
    }
    
    if (io.botao_m2_on && relogio.aciona_horario == 0) //AND com relogio para garantir que quando o relogio esteja ativado, não seja possível acionar botao
    {
      contador_botao_m2++;
      if(contador_botao_m2 > TEMPO_DEBOUCING_MOTOR)
      {
        contador_botao_m2  = 0;
        io.botao_m2_on     = 0;
        
        io.buzzer_on       = 1;            //para ativar buzzer
        io.buzzer_tipo     = BEEP_SIMPLES; //para definir o beep do motor 
        io.motor_on        = 1;            //flag de MOTOR ON
        io.numero_motor    = MOTOR_2;      //para definir botao 1 = motor 1
        io.aciona_motor_2_horario = 1;     //para ligar motor 1 
      }
    }
      else
      {
        //io.buzzer_on = 0;
      }
    }


    void trata_botao(void)
    {
    if (io.botao_menu_aciona)
    {
      if (lcd.tela <= INFOS)
      {
        lcd.tela++;
      }

      if (lcd.tela > INFOS)
      {
        lcd.tela = INICIO;      //INFOS é a última tela
      }
    }
    }

    void atualiza_display(void)
    {
      lcd.nivel = 0; //apenas um nivel de menu...
    switch (lcd.nivel)
    {
      case 0:

        switch (lcd.tela) //usar tela com botao
        {
          case INICIO:
            display.clearDisplay();
            display.setTextSize(1.9);             // definição tamanho de letra para display
            display.setTextColor(WHITE);          // definição de cor de letra para display
            //display.setRotation(2);             // definição de rotação de letra para display
            display.setCursor(28, 0);             // nessa posicao = letra amarela (x,y)
            display.println("S. EMBARCADOS");
            display.setCursor(5, 18);
            display.print("Dispenser de Remedio");
            display.setCursor(0, 40);
            display.print("Autor:" + autorName);
            display.setCursor(0, 55);
            display.print("RA: 20.01071-0");
            display.display();
            break;

          case HORARIO:
            display.clearDisplay();
            display.setTextSize(2);               // definição tamanho de letra para display
            display.setTextColor(WHITE);          // definição de cor de letra para display
            display.setCursor(20, 0);             // nessa posicao = letra amarela (x,y)
            display.println(relogio.formata_msg);
            display.setTextSize(1.9);
            display.setCursor(0, 40);
            display.print("1. Horario:");
            display.print(relogio.primeiro_horario);
            display.setCursor(0, 55);
            display.print("2. Horario:");
            display.print(relogio.segundo_horario);
            display.display();
            break;

          case INFOS:
            display.clearDisplay();
            display.setTextSize(1.9);             // definição tamanho de letra para display
            display.setTextColor(WHITE);          // definição de cor de letra para display
            display.setCursor(25, 0);             // nessa posicao = letra amarela (x,y)
            display.println("INFOS TECNICAS");
            display.setCursor(0, 25);             
            display.println("HW/SW:" + autorName);
            display.setCursor(0, 40);             
            display.println("Versao:" + Version);
            display.setCursor(0, 55);             
            display.println("Data:" + dataVersion);
            display.display();
            break;
        }
        break;
    }
    }

    void trata_horario(void)
    {
    sprintf(relogio.primeiro_horario, "%02d:%02d:%02d", 20, 18, 00); // montando string com char
    sprintf(relogio.segundo_horario,  "%02d:%02d:%02d", 20, 20, 00); // montando string com char
  
    relogio.primeiro_horario_desejado = strcmp(relogio.primeiro_horario, relogio.formata_msg); //se for igual, retorna 0
    relogio.segundo_horario_desejado  = strcmp(relogio.segundo_horario,  relogio.formata_msg); //se for igual, retorna 0

    if(!relogio.primeiro_horario_desejado)
    {
    io.motor_on = 1;
    io.numero_motor = MOTOR_1;
    io.aciona_motor_1_horario = 1;
    }
     if(!relogio.segundo_horario_desejado)
    {
    io.motor_on = 1;
    io.numero_motor = MOTOR_2;
    io.aciona_motor_2_horario = 1;
    }
    }

    void trata_motor(void)
    {
    static unsigned int contador_motor_1_horario_on  = 0; //static mantém valor da variável
    static unsigned int contador_motor_1_pausa       = 0; //static mantém valor da variável
    static unsigned int contador_motor_1_anti_on     = 0; //static mantém valor da variável
    static unsigned int contador_motor_2_horario_on  = 0; //static mantém valor da variável
    static unsigned int contador_motor_2_pausa       = 0; //static mantém valor da variável
    static unsigned int contador_motor_2_anti_on     = 0; //static mantém valor da variável

    if(io.motor_on)
    {
    switch(io.numero_motor)
    {
    case MOTOR_1:                     //aciona motor 1

      if(io.aciona_motor_1_horario)
      {
      contador_motor_1_horario_on ++;  //tempo para motor girar no sentido horario
      digitalWrite(MOTOR_1A, HIGH);
      digitalWrite(MOTOR_1B, LOW);
      
      if(contador_motor_1_horario_on > TEMPO_MOTOR_ON)
      {
         contador_motor_1_horario_on = 0;
         io.aciona_motor_1_horario   = 0;
         io.aciona_motor_1_pausa     = 1;
         digitalWrite(MOTOR_1A, LOW);
         digitalWrite(MOTOR_1B, LOW);
      }
    }
      if(io.aciona_motor_1_pausa)
      {
        contador_motor_1_pausa++; //tempo de motor off entre abrir e fechar
        if(contador_motor_1_pausa > TEMPO_MOTOR_PAUSA)
        {
          contador_motor_1_pausa  = 0;
          io.aciona_motor_1_pausa = 0;
          io.aciona_motor_1_anti  = 1;
         digitalWrite(MOTOR_1A,  LOW);
         digitalWrite(MOTOR_1B, HIGH);          
        }
      }
      if(io.aciona_motor_1_anti)
      {
        contador_motor_1_anti_on++;  //tempo para motor girar no sentido anti_horário (deve ser igual ao horario)
        if(contador_motor_1_anti_on > TEMPO_MOTOR_ON)
        {
          contador_motor_1_anti_on = 0;
          io.aciona_motor_1_anti = 0;
          digitalWrite(MOTOR_1A,  LOW);
          digitalWrite(MOTOR_1B,  LOW);     
        }  
      }
     break;

     case MOTOR_2:                     //aciona motor 2
      if(io.aciona_motor_2_horario)
      {
      contador_motor_2_horario_on ++;  //tempo para motor girar no sentido anti horario
      digitalWrite(MOTOR_2A, HIGH);
      digitalWrite(MOTOR_2B, LOW);
      
      if(contador_motor_2_horario_on > TEMPO_MOTOR_ON)
      {
         contador_motor_2_horario_on = 0;
         io.aciona_motor_2_horario   = 0;
         io.aciona_motor_2_pausa     = 1;
         digitalWrite(MOTOR_2A, LOW);
         digitalWrite(MOTOR_2B, LOW);
      }
    }
      if(io.aciona_motor_2_pausa)
      {
        contador_motor_2_pausa++; //tempo de motor off entre abrir e fechar
        if(contador_motor_2_pausa > TEMPO_MOTOR_PAUSA)
        {
          contador_motor_2_pausa  = 0;
          io.aciona_motor_2_pausa = 0;
          io.aciona_motor_2_anti  = 1;
          digitalWrite(MOTOR_2A,  LOW);
          digitalWrite(MOTOR_2B, HIGH);          
        }
      }
      if(io.aciona_motor_2_anti)
      {
        contador_motor_2_anti_on++;  //tempo para motor girar no sentido horário (deve ser igual ao horario)
        if(contador_motor_2_anti_on > TEMPO_MOTOR_ON)
        {
          contador_motor_2_anti_on = 0;
          io.aciona_motor_2_anti   = 0;
          digitalWrite(MOTOR_2A,  LOW);
          digitalWrite(MOTOR_2B,  LOW);     
        }  
      }
     break;
    }
    }
    }

# Biblioteca Criada

    #include <Wire.h>
    #include <WiFi.h>
    #include <PubSubClient.h>
    #include <Adafruit_GFX.h>
    #include <Adafruit_SSD1306.h>

    #include "RTClib.h"      //Módulo RTC DS3231

    // display OLED DEFINIÇÕES
    #define SCREEN_WIDTH 128 // OLED display width, in pixels
    #define SCREEN_HEIGHT 64 // OLED display height, in pixels
    // Declaration for an SSD1306 display connected to I2C (SDA, SCL pins)
    #define OLED_RESET     -1 // Reset pin # (or -1 if sharing Arduino resetpin)
    Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

    //definição de pinos
    #define BUZZER_PIN        4     //buzzer
    #define MOTOR_1A          33    //motor A-1A
    #define MOTOR_1B          25    //motor A-1B
    #define MOTOR_2A          26    //motor B-2A
    #define MOTOR_2B          27    //motor B-2B
  
    //definição de pinos dos botões
    #define BUTTON_MENU_PIN      35    //botão de menu
    #define BUTTON_M2_PIN        34    //botão do motor 1
    #define BUTTON_M1_PIN        32    //botão do motor 2

    //definição DEBOUCING dos BOTÕES
    #define TEMPO_DEBOUCING_MENU  250
    #define TEMPO_DEBOUCING_MOTOR 1000

    //definição tempo BEEP do buzzer
    #define TEMPO_SIMPLES         110
    #define TEMPO_DUPLO_CURTO     160   //TEMPO DO PRIMEIRO BEEP do duplo
    #define TEMPO_DUPLO_AUX_LOW   350   //TEMPO DE MUDO pós primeiro BEEP
    #define TEMPO_DUPLO_AUX_HIGH  600   //TEMPO DO SEGUNDO BEEP do duplo
    #define TEMPO_BEEP_MOTOR      400   //TEMPO DO BEEP QUANDO MOTOR ON

    //definição tempo  dos MOTORES 
    #define TEMPO_MOTOR_ON    300   //TEMPO DE MOTOR PARA ABRIR E FECHAR
    #define TEMPO_MOTOR_PAUSA 2000  //TEMPO DE MOTOR OFF DURANTE ABERTURA E FECHADURA


    typedef struct
    {
    bool buzzer_inicializa;
    bool buzzer_on;
    bool buzzer_simples_on;
    bool buzzer_duplo_on;
    int  buzzer_tipo;
    bool buzzer_tempo_duplo_low;
    bool buzzer_tempo_duplo_high;
    bool buzzer_motor;
  
    bool botao_menu_on;
    bool botao_menu_aciona; 
    bool botao_m1_on;
    bool botao_m2_on;

    int numero_motor;
    int motor_on;
    int motor_tipo;
    int aciona_motor_1_horario;
    int aciona_motor_1_pausa;
    int aciona_motor_1_anti;
    int aciona_motor_2_horario;
    int aciona_motor_2_pausa;
    int aciona_motor_2_anti;
    }Io;

    typedef struct
    {
    int segundos;
    int minutos;
    int hora;
    char formata_msg [50];
    char primeiro_horario [50];
    char segundo_horario [50];
    int primeiro_horario_desejado;
    int segundo_horario_desejado;
    bool aciona_horario;
    //bool aciona_horario;
    }Relogio;

    typedef struct
    {
    int nivel;
    int tela;
    }Lcd;

    typedef struct
    {
    bool GREEN_on;
    bool RED_on;
    }Led;

     typedef enum  //referenciar numero com palavra
     {
    INICIO  = 0,
    HORARIO = 1,
    INFOS   = 2,
     }Tela;

    typedef enum  //referenciar numero com palavra
     {
    BEEP_SIMPLES = 0,
    BEEP_DUPLO   = 1,
    BEEP_MOTOR   = 2,
     }Buzzer;

     typedef enum  //referenciar numero com palavra
     {  
    MOTOR_OFF       = 0,
    MOTOR_HORARIO   = 1,
    MOTOR_ANTI      = 2,
    MOTOR_OFF2      = 3,
    MOTOR_1         = 4,
    MOTOR_2         = 5,
    MOTOR_AMBOS     = 6,
     }Motor;

 

