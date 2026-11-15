module tb_alu;
  reg  [15:0] a, b;
  reg  [2:0]  op;
  wire [15:0] result;
  wire        zero;

  alu dut (.a(a), .b(b), .op(op), .result(result), .zero(zero));

  task check;
    input [15:0] expected;
    begin
      #10;
      if (result === expected)
        $display("PASS: op=%b a=%d b=%d result=%d", op, a, b, result);
      else
        $display("FAIL: op=%b a=%d b=%d result=%d expected=%d", op, a, b, result, expected);
    end
  endtask

  initial begin
	$dumpfile("dump.vcd");
    $dumpvars(0, tb_alu);
    a = 5;  b = 3;  op = 3'b000; check(16'd8);      
    a = 5;  b = 3;  op = 3'b001; check(16'd2);      
    a = 12; b = 10; op = 3'b010; check(16'd8);      
    a = 12; b = 10; op = 3'b011; check(16'd14);     
    a = 12; b = 10; op = 3'b100; check(16'd6);      
    a = 5;  b = 0;  op = 3'b101; check(16'd10);     
    a = 5;  b = 0;  op = 3'b110; check(16'd2);      
    a = 3;  b = 5;  op = 3'b111; check(16'd1);      
    a = 5;  b = 3;  op = 3'b111; check(16'd0);      

    
    a = 3;  b = 5;  op = 3'b001; check(16'd65534);  
    a = 5;  b = 5;  op = 3'b001; check(16'd0);      
    $display("zero flag = %b (should be 1)", zero);

    $finish;
  end
endmodule