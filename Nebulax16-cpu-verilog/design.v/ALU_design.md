module alu (

&#x20; input  \[15:0] a, b,

&#x20; input  \[2:0]  op,

&#x20; output reg \[15:0] result,

&#x20; output zero

);

&#x20; always @(\*) begin

&#x20;   case (op)

&#x20;     3'b000: result = a + b;

&#x20;     3'b001: result = a - b;

&#x20;     3'b010: result = a \& b;

&#x20;     3'b011: result = a | b;

&#x20;     3'b100: result = a ^ b;

&#x20;     3'b101: result = a << 1;

&#x20;     3'b110: result = a >> 1;

&#x20;     3'b111: result = (a < b) ? 16'd1 : 16'd0;

&#x20;     default: result = 16'd0;

&#x20;   endcase

&#x20; end

&#x20; assign zero = (result == 16'd0);

endmodule

