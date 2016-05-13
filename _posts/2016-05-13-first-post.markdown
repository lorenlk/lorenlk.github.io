---
layout: post
title:  "First Post!"
date:   2016-05-13 18:30:00 -0400
categories: general
---
This first post will be to try the highlight capability of the jekyll environment.

This is a VHDL implementation for an ALU.

{% highlight vhdl %}

-- ALU

library ieee;
use ieee.std_logic_1164.all;
use ieee.std_logic_arith.all;
use ieee.std_logic_unsigned.all;

use work.instruct_pkg.all;

entity ALU is
	port (
		A, B        : in std_logic_vector(15 downto 0);
		OP_CODE     : in std_logic_vector(5 downto 0);
        C, Z        : out std_logic;
        S           : out std_logic_vector(15 downto 0)
	);
end ALU;

architecture hardware of ALU is

	signal sumA, sumB, res : std_logic_vector (15 downto 0);
    signal sgn : std_logic_vector (8 downto 0);
    signal n : integer;
    signal C_SHLN, C_SHLAN, C_SHRN, C_RTL, C_RTR, C_INC, C_DEC, C_ADD, C_SUB : std_logic;
    signal Z_RES, Z_AND, Z_OR, Z_XOR : std_logic;
    signal S_RST, S_SET, S_SHLN, S_SHRN, S_RTL, S_RTR, S_AND, S_OR, S_XOR, S_RES : std_logic_vector (15 downto 0);
    signal S_AUX: std_logic_vector (31 downto 0);
    signal usumA, usumB : unsigned (15 downto 0);
    
begin
  
	-- Operation Code Decodification.
    
    -- Carry out
    with OP_CODE select    
    C <=    C_SHLN when SHLN,
            C_SHLAN when SHLAN,
            C_SHRN when SHRN,
            C_RTL  when RTL,
            C_RTR  when RTR,
            C_INC  when INC,
            C_DEC  when DEC,
            C_ADD  when OP_ADD,
            C_SUB  when OP_SUB,
            '1'    when SETC,
            '0'    when others;
    
    
    -- Zero out
    with OP_CODE select    
    Z <=    '1'   when SETZ,
            '1'   when RST,
            '0'   when CLRZ,
            '0'   when SET,
            Z_RES when SHLAN,
            Z_RES when SHRAN,
            Z_RES when INC,
            Z_RES when DEC,
            Z_RES when OP_ADD,
            Z_RES when OP_SUB,
            Z_RES when BITSET,
            Z_RES when BITCLR,
            Z_AND when OP_AND,
            Z_OR  when OP_OR,
            Z_XOR when OP_XOR,
            Z_XOR when INV,
            '0'    when others;
    
    -- Output
    with OP_CODE select    
    S <=    S_RST   when RST,
            S_SET   when SET,
            S_SHLN  when SHLN,
            S_SHRN  when SHRN,
            S_RTL   when RTL,
            S_RTR   when RTR,
            res     when INC,
            res     when DEC,
            res     when OP_ADD,
            res     when OP_SUB,
            res     when MUL,
            S_AND   when OP_AND,
            S_OR    when OP_OR,
            S_XOR   when OP_XOR,
            S_XOR   when INV,
            res     when BITSET,
            res     when BITCLR,
            x"0000" when others;
          
    -- Aux. signal A
    with OP_CODE select    
    sumA <= A   when SHLN,
            A   when SHLAN,
            A   when SHRN,
            A   when SHRAN,
            A   when RTL,
            A   when RTR,
            A   when INC,
            A   when DEC,
            A   when OP_ADD,
            A   when OP_SUB,
            A   when MUL,
            A   when OP_AND,
            A   when OP_OR,
            A   when INV,
            A   when OP_XOR,
            A   when BITSET,
            A   when BITCLR,
            x"0000" when others;
    
    -- Aux. signal B
    with OP_CODE select    
    sumB <= B     when SHLN,
            B     when SHLAN,
            B     when SHRN,
            B     when SHRAN,
            B     when OP_ADD,
            B     when OP_SUB,
            B     when MUL,
            B     when OP_AND,
            B     when OP_OR,
            B     when OP_XOR,
            B     when BITSET,
            B     when BITCLR,
            x"0001" when INC,
            x"FFFF" when DEC,
            x"FFFF" when INV,
            x"0000" when others;
    
    -- Number of bits to shift
    n <= conv_integer(sumB(2 downto 0));
            
    -- ALU operations
    
    -- Carry out
    with n select    
    C_SHLN <= sumA(NBITS - 1) when 1,
              sumA(NBITS - 2) when 2,
              sumA(NBITS - 3) when 3,
              sumA(NBITS - 4) when 4,
              sumA(NBITS - 5) when 5,
              sumA(NBITS - 6) when 6,
              sumA(NBITS - 7) when 7,
              sumA(NBITS - 8) when others;
    
    with n select
    C_SHLAN <= sumA(NBITS - 2) when 1,
               sumA(NBITS - 3) when 2,
               sumA(NBITS - 4) when 3,
               sumA(NBITS - 5) when 4,
               sumA(NBITS - 6) when 5,
               sumA(NBITS - 7) when 6,
               sumA(NBITS - 8) when 7,
               sumA(NBITS - 9) when others; 
        
    with n select    
    C_SHRN <= sumA(0) when 1,
              sumA(1) when 2,
              sumA(2) when 3,
              sumA(3) when 4,
              sumA(4) when 5,
              sumA(5) when 6,
              sumA(6) when 7,
              sumA(7) when others;

    C_RTL <= sumA(NBITS - 1);
    
    C_RTR <= sumA(0);
    
    with sumA select    
    C_INC <= '1' when x"7FFF",
             '0' when others;
    
    with sumA select    
    C_DEC <= '1' when x"8000",
             '0' when others;
    
    carry_add: process (sumA, sumB, res)
    begin    
        C_ADD <= '0';
        
        if ((sumA(15) = '0' AND sumB(15) = '0' AND res(15) = '1')
            OR (sumA(15) = '1' AND sumB(15) = '1' AND res(15) = '0')) then
            
            C_ADD <= '1';
            
        end if;
        
    end process;
    
    carry_sub: process (sumA, sumB, res)
    begin    
        C_SUB <= '0';
        
        if ((sumA(15) = '1' AND sumB(15) = '0' AND res(15) = '0')
            OR (sumA(15) = '0' AND sumB(15) = '1' AND res(15) = '1')) then
            
            C_ADD <= '1';
            
        end if;
        
    end process;
    
    -- Zero out
    
    with res select
    Z_RES <= '1' when x"0000",
             '0' when others;
    
    with S_AND select
    Z_AND <= '1' when x"0000",
             '0' when others;
             
    with S_OR select
    Z_OR <= '1' when x"0000",
             '0' when others;
    
    with S_XOR select
    Z_XOR <= '1' when x"0000",
             '0' when others;
             
    -- S out
    
    S_RST <= x"0000";
    
    S_SET <= x"FFFF";
    
    sgn <= (others => sumA(NBITS - 1));
    
    with n select 
    S_SHLN <= sumA(NBITS - 2 downto 0) & "0" when 0,
              sumA(NBITS - 3 downto 0) & "00" when 1,
              sumA(NBITS - 4 downto 0) & "000" when 2,
              sumA(NBITS - 5 downto 0) & "0000" when 3,
              sumA(NBITS - 6 downto 0) & "00000" when 4,
              sumA(NBITS - 7 downto 0) & "000000" when 5,
              sumA(NBITS - 8 downto 0) & "0000000" when 6,
              sumA(NBITS - 9 downto 0) & "00000000" when 7,
              sumA when others;
    
    with n select
    S_SHLAN <= sgn(0) & sumA(NBITS - 3 downto 0) & "0" when 0,
               sgn(0) & sumA(NBITS - 4 downto 0) & "00" when 1,
               sgn(0) & sumA(NBITS - 5 downto 0) & "000" when 2,
               sgn(0) & sumA(NBITS - 6 downto 0) & "0000" when 3,
               sgn(0) & sumA(NBITS - 7 downto 0) & "00000" when 4,
               sgn(0) & sumA(NBITS - 8 downto 0) & "000000" when 5,
               sgn(0) & sumA(NBITS - 9 downto 0) & "0000000" when 6,
               sgn(0) & sumA(NBITS - 10 downto 0) & "00000000" when 7,
               sumA when others;
            
    with n select 
    S_SHRN <= "1" & sumA(NBITS - 1 downto 1) when 0,
              "11" & sumA(NBITS - 1 downto 2) when 1,
              "111" & sumA(NBITS - 1 downto 3) when 2,
              "1111" & sumA(NBITS - 1 downto 4) when 3,
              "11111" & sumA(NBITS - 1 downto 5) when 4,
              "111111" & sumA(NBITS - 1 downto 6) when 5,
              "1111111" & sumA(NBITS - 1 downto 7) when 6,
              "11111111" & sumA(NBITS - 1 downto 8) when 7,
              sumA when others;
    
    with n select
    S_SHRAN <= sgn(1 downto 0) & sumA(NBITS - 3 downto 0) when 0,
               sgn(2 downto 0) & sumA(NBITS - 4 downto 0) when 1,
               sgn(3 downto 0) & sumA(NBITS - 5 downto 0) when 2,
               sgn(4 downto 0) & sumA(NBITS - 6 downto 0) when 3,
               sgn(5 downto 0) & sumA(NBITS - 7 downto 0) when 4,
               sgn(6 downto 0) & sumA(NBITS - 8 downto 0) when 5,
               sgn(7 downto 0) & sumA(NBITS - 9 downto 0) when 6,
               sgn(8 downto 0) & sumA(NBITS - 10 downto 0) when 7,
               sumA when others;
    
    S_RTL <= sumA(NBITS - 2 downto 0) & sumA(NBITS - 1);
    
    S_RTR <= sumA(0) & sumA(NBITS - 1 downto 1);
    
    arithmetic: process (sumA, sumB, res, OP_CODE, S_AUX)
    begin
        res <= x"0000";
        S_AUX <= x"00000000";
        
        if (OP_CODE = INC OR OP_CODE = DEC OR OP_CODE = NEG) then
        
            res <= sumA + sumB;
        
        elsif (OP_CODE = OP_ADD OR OP_CODE = OP_SUB) then
        
            S_AUX <= sumA + sumB;
            
            if (S_AUX(16) = '1') then
            
                res <= x"FFFF";
                
            else
            
                res <= S_AUX (15 downto 0);
            end if;
          
        elsif (OP_CODE = MUL) then
        
            S_AUX <= sumA * sumB;
            res <= S_AUX (15 downto 0);
        
        elsif (OP_CODE = BITSET) then
        
            sumA(n) <= '1';
            res <= sumA;
            
        elsif (OP_CODE = BITCLR) then
        
            sumA(n) <= '0';
            res <= sumA;
        
        end if;
    end process;
    
    S_AND <= sumA AND sumB;
    
    S_OR <= sumA OR sumB;
    
    S_XOR <= sumA XOR sumB; -- Do both XOR and INV operations
        
end hardware;

{% endhighight %}

Check out [ALU][alu-def] for further information.

[alu-def]: https://en.wikipedia.org/wiki/Arithmetic_logic_unit