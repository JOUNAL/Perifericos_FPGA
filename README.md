# Perifericos_FPGA
En este repositorio se seguira el paso a paso de como manipular y controlar un servomotor a travez de las señales logicas obtenidas a travez de la FPGA

<h3>1).Definir el comportamineto</h3>
A continuacion se presenta el diagrma de flujo que contendria el control de un servo a travez de las operaciones realizables para una FPGA
<img src="https://github.com/JOUNAL/Perifericos_FPGA/blob/main/Imagenes/flujograma_servo.png" width=80% height=80%>


<h3>2).Definir la estructura</h3>
A continuacion se presenta el diagrma de caja negra y las conexiones fisicas con la FPGA que se tendria para el control de un servomotor
<img src="https://github.com/JOUNAL/Perifericos_FPGA/blob/main/Imagenes/Servo_caja_negra.png" width=50% height=50%>

<h3>3).Describir el diseño en HDL</h3>
A continuacion se presenta el codigo extraido a travez de la interpretacion del diagrama de flujo en verilog

```VERILOG
module top(
    input wire clk,
    input wire [1:0] sw,      // sw[0] = PIN 86, sw[1] = PIN 98
    output reg servo_pwm
);
    // 1. GENERADOR PWM (Periodo exacto de 20ms = 1,000,000 ciclos a 50 MHz)
    reg [19:0] contador_pwm = 20'd0;
    reg [19:0] pulso_actual = 20'd25000; // Arranca por defecto en 0 grados

    always @(posedge clk) begin
        if (contador_pwm >= 20'd999999) begin
            contador_pwm <= 20'd0;
        end else begin
            contador_pwm <= contador_pwm + 1'b1;
        end

        // Salida de la señal PWM hacia el pin del servo
        if (contador_pwm < pulso_actual) begin
            servo_pwm <= 1'b1;
        end else begin
            servo_pwm <= 1'b0;
        end
    end

    // 2. CONTROL POR SWITCHES (Adaptado a Lógica Inversa de Pull-Up)
    always @(posedge clk) begin
        case (sw)
            2'b11: pulso_actual <= 20'd25000;  // Ambos abiertos (Físicamente en 0,0) -> 0 Grados
            2'b10: pulso_actual <= 20'd58333;  // sw[0] cerrado   (Físicamente en 0,1) -> 60 Grados 
            2'b01: pulso_actual <= 20'd91666;  // sw[1] cerrado   (Físicamente en 1,0) -> 120 Grados 
            2'b00: pulso_actual <= 20'd125000; // Ambos cerrados  (Físicamente en 1,1) -> 180 Grados 
            default: pulso_actual <= 20'd25000;
        endcase
    end

endmodule
```

<h3>4).Representacion en RTL</h3>
A continuacion se presenta la representacion en RTL
<img src="https://github.com/JOUNAL/Perifericos_FPGA/blob/main/Imagenes/RTL_SERVO.png" width=70% height=70%>
