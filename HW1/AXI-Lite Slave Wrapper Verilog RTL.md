````vhdl
library ieee;
use ieee.std_logic_1164.all;
use ieee.numeric_std.all;

entity axi_uart_wrapper is
    port (
        clk           : in  std_logic;
        rst_n         : in  std_logic;
        
        -- AXI-Lite 介面簡化
        s_axi_awaddr  : in  std_logic_vector(31 downto 0);
        s_axi_awvalid : in  std_logic;
        s_axi_awready : out std_logic;
        s_axi_wdata   : in  std_logic_vector(31 downto 0);
        s_axi_wvalid  : in  std_logic;
        s_axi_wready  : out std_logic;
        s_axi_rdata   : out std_logic_vector(31 downto 0);
        s_axi_rvalid  : out std_logic;
        s_axi_rready  : in  std_logic
    );
end entity axi_uart_wrapper;

architecture rtl of axi_uart_wrapper is

    -- 狀態定義
    type state_type is (STATE_IDLE, STATE_WRITE, STATE_READ);
    signal state : state_type;

    -- 模擬內建 UART 暫存器
    signal uart_tx_buff : std_logic_vector(31 downto 0);

begin

    process(clk, rst_n)
    begin
        if rst_n = '0' then
            state         <= STATE_IDLE;
            s_axi_awready <= '0';
            s_axi_wready  <= '0';
            s_axi_rvalid  <= '0';
            uart_tx_buff  <= (others => '0');
            s_axi_rdata   <= (others => '0');
        elsif rising_edge(clk) then
            case state is
                when STATE_IDLE =>
                    if (s_axi_awvalid = '1' and s_axi_wvalid = '1') then
                        s_axi_awready <= '1';
                        s_axi_wready  <= '1';
                        state         <= STATE_WRITE;
                    end if;
                    
                when STATE_WRITE =>
                    uart_tx_buff  <= s_axi_wdata; -- 寫入 UART Buff
                    s_axi_awready <= '0';
                    s_axi_wready  <= '0';
                    state         <= STATE_IDLE;
                    
                when others =>
                    state <= STATE_IDLE;
            end case;
        end if;
    end process;

end architecture rtl;
````
