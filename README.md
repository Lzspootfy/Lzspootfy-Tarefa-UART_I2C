# Introdução às Interfaces de Comunicação Serial com RP2040

## UART, SPI e I2C

Repositório criado a fim de armazenar a tarefa realizada para consolidar a compreensão dos conceitos sobre o uso de interfaces de comunicação serial no RP2040 e explorar as funcionalidades da placa de desenvolvimento BitDogLab.

▶️Video demonstarção https://youtu.be/uBmhBs05Tvw
---

## Objetivos 🎯

- Compreender o funcionamento e a aplicação de comunicação serial em microcontroladores;
- Aplicar os conhecimentos adquiridos sobre UART e I2C na prática;
- Manipular e controlar LEDs comuns e LEDs endereçáveis WS2812;
- Fixar o estudo do uso botões de acionamento, interrupções e debounce;
- Desenvolver um projeto funcional que combine hardware e software.

---

## Descrição do Projeto 🛠️

### Utilização obrigatória:

- Matriz 5x5 de LEDs (endereçáveis) WS2812, conectada à GPIO 7;
- LED RGB, com os pinos conectados às GPIOs (11, 12 e 13);
- Botão A conectado à GPIO 5;
- Botão B conectado à GPIO 6;
- Display SSD1306 conectado via I2C (GPIO 14 e GPIO 15).

---

## Funcionalidades do Projeto 🚀

### 1️⃣ Modificação da Biblioteca `font.h`
- Adicionar caracteres minúsculos à biblioteca `font.h`.

### 2️⃣ Entrada de caracteres via PC 🖥️
- Utilize o Serial Monitor do VS Code para digitar os caracteres.
- Cada caractere digitado no Serial Monitor deve ser exibido no display SSD1306.
- Quando um número entre 0 e 9 for digitado, um símbolo correspondente deve ser exibido na matriz 5x5 WS2812.

### 3️⃣ Interação com o Botão A 🔘
- Pressionar o botão A deve alternar o estado do LED RGB Verde (ligado/desligado).
- O estado do LED deve ser exibido no display SSD1306 e uma mensagem informativa enviada ao Serial Monitor.

### 4️⃣ Interação com o Botão B 🔘
- Pressionar o botão B deve alternar o estado do LED RGB Azul (ligado/desligado).
- O estado do LED deve ser exibido no display SSD1306 e uma mensagem informativa enviada ao Serial Monitor.

---

## Requisitos do Projeto 📌

### 1️⃣ Uso de Interrupções (IRQ) para Botões ⚡

```c
gpio_set_irq_enabled_with_callback(BUTTON_A, GPIO_IRQ_EDGE_FALL, true, &gpio_irq_callback);
gpio_set_irq_enabled_with_callback(BUTTON_B, GPIO_IRQ_EDGE_FALL, true, &gpio_irq_callback);
```

A função `gpio_irq_callback` é usada para tratar as interrupções e alternar o estado dos LEDs:

```c
void gpio_irq_callback(uint gpio, uint32_t events) {
  uint32_t current_time = to_ms_since_boot(get_absolute_time());
  if (current_time - last_time > 200) { // Debouncing de 200ms
    last_time = current_time;
    if (gpio == BUTTON_A) {
      led_green_state = !led_green_state;
      gpio_put(GREEN_RGB, led_green_state);
      printf("Botão A pressionado mudou o estado do LED verde para %s\n", led_green_state ? "1" : "0");
    } else if (gpio == BUTTON_B) {
      led_blue_state = !led_blue_state;
      gpio_put(BLUE_RGB, led_blue_state);
      printf("Botão B pressionado mudou o estado do LED azul para %s \n", led_blue_state ? "1" : "0");
    }
  }
}
```

### 2️⃣ Debouncing via Software ⏳

```c
if (current_time - last_time > 200) { // Debouncing de 200ms
```

Isso evita múltiplas detecções devido ao efeito de bouncing dos botões.

### 3️⃣ Controle de LEDs (Comuns e WS2812) 💡

**LEDs Comuns:**

```c
gpio_init(RED_RGB);
gpio_init(GREEN_RGB);
gpio_init(BLUE_RGB);
gpio_set_dir(RED_RGB, GPIO_OUT);
gpio_set_dir(GREEN_RGB, GPIO_OUT);
gpio_set_dir(BLUE_RGB, GPIO_OUT);
gpio_put(RED_RGB, 0);
gpio_put(GREEN_RGB, 0);
gpio_put(BLUE_RGB, 0);
```

**LEDs WS2812:**

```c
npInit(LED_PIN);
npClear();
npWrite();
```

### 4️⃣ Utilização do Display 128x64 via I2C 🖥️

```c
i2c_init(I2C_PORT, 400 * 1000);
gpio_set_function(I2C_SDA, GPIO_FUNC_I2C);
gpio_set_function(I2C_SCL, GPIO_FUNC_I2C);
gpio_pull_up(I2C_SDA);
gpio_pull_up(I2C_SCL);
ssd1306_init(ssd, WIDTH, HEIGHT, false, ENDERECO, I2C_PORT);
ssd1306_fill(ssd, false);
ssd1306_send_data(ssd);
```

Atualização do display:

```c
ssd1306_draw_string(&ssd, string_a, 8, 40);
ssd1306_draw_string(&ssd, string_b, 8, 48);
ssd1306_send_data(&ssd);
```

### 5️⃣ Envio de Informação pela UART 🔄

```c
uart_init(UART_ID, 115200);
gpio_set_function(0, GPIO_FUNC_UART);
gpio_set_function(1, GPIO_FUNC_UART);
```

Recebe caracteres via `getc(stdin)` e exibe no display:

```c
if(stdio_usb_connected) {
  c[0] = getc(stdin);
  handle_numbers(c[0]);
  ssd1306_draw_string(&ssd, "Caractere: " , 8, 16);
  ssd1306_draw_string(&ssd, c, 80, 16);
  ssd1306_send_data(&ssd);
}
```

