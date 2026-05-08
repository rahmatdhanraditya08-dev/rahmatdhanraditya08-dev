#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
INDUSTRIAL COOL DASHBOARD - RADITYA RAHMAT
Real-time terminal HUD untuk monitoring proses
"""

import random
import time
import threading
import os
import sys
from datetime import datetime

try:
    from rich.console import Console
    from rich.panel import Panel
    from rich.layout import Layout
    from rich.live import Live
    from rich.table import Table
    from rich.progress_bar import ProgressBar
    import asciichartpy as acp
except ImportError:
    print("Instal dulu: pip install rich asciichartpy")
    sys.exit(1)

console = Console()

# Data simulasi sensor
class SensorSim:
    def __init__(self):
        self.temp = 75.0      # °C
        self.pressure = 2.3   # bar
        self.humidity = 45.0   # %
        self.vibration = 0.12  # mm/s
        self.trend = []
        for _ in range(50):
            self.trend.append(random.uniform(70, 80))
    
    def update(self):
        # perubahan random realistis
        self.temp += random.uniform(-0.5, 0.8)
        self.pressure += random.uniform(-0.05, 0.07)
        self.humidity += random.uniform(-0.8, 0.9)
        self.vibration = max(0, self.vibration + random.uniform(-0.02, 0.03))
        # batas rasional
        self.temp = max(60, min(95, self.temp))
        self.pressure = max(1.8, min(3.2, self.pressure))
        self.humidity = max(30, min(70, self.humidity))
        self.vibration = max(0.05, min(0.45, self.vibration))
        
        # update trend line
        self.trend.append(self.temp)
        if len(self.trend) > 50:
            self.trend.pop(0)
    
    def get_chart(self):
        return acp.plot(self.trend, {'height': 8, 'width': 40, 'colors': [acp.green]})

def matrix_effect_line():
    """Cetak satu baris efek matrix random"""
    chars = "01"
    line = ''.join(random.choice(chars) for _ in range(60))
    return f"[bold green]{line}[/bold green]"

def create_layout(sensor, counter):
    # Layout utama
    layout = Layout()
    layout.split(
        Layout(name="header", size=3),
        Layout(name="main"),
        Layout(name="footer", size=4)
    )
    layout["main"].split_row(
        Layout(name="left", ratio=2),
        Layout(name="right", ratio=1)
    )
    layout["left"].split_column(
        Layout(name="sensors"),
        Layout(name="trend", size=12)
    )
    layout["right"].split_column(
        Layout(name="status"),
        Layout(name="extra")
    )
    
    # HEADER dengan animasi matrix
    header_text = f"""
[bold cyan]╔════════════════════════════════════════════════════════════════╗
║            RADITYA RAHMAT - INDUSTRIAL CONTROL SYSTEM           ║
║         [green]>> REAL-TIME MONITORING v2.1 <<[/green]                   ║
╚════════════════════════════════════════════════════════════════╝[/bold cyan]
{matrix_effect_line()}
    """
    layout["header"].update(Panel(header_text, title="[red] SYSTEM ONLINE [/red]", border_style="green"))
    
    # Panel sensor utama
    sensor_table = Table(show_header=True, header_style="bold cyan", box=None)
    sensor_table.add_column("Parameter", style="dim")
    sensor_table.add_column("Nilai", justify="right")
    sensor_table.add_column("Status", justify="center")
    
    temp_bar = ProgressBar(total=100, width=20, style="green", completed=sensor.temp-60 if sensor.temp>60 else 0)
    press_bar = ProgressBar(total=3.2, width=20, style="yellow", completed=sensor.pressure)
    vib_bar = ProgressBar(total=0.5, width=20, style="red", completed=sensor.vibration)
    
    sensor_table.add_row("🌡️ Suhu Reaktor", f"{sensor.temp:.1f} °C", str(temp_bar))
    sensor_table.add_row("⏲️ Tekanan", f"{sensor.pressure:.2f} bar", str(press_bar))
    sensor_table.add_row("💧 Kelembaban", f"{sensor.humidity:.1f} %", f"[cyan]{sensor.humidity:.0f}%[/cyan]")
    sensor_table.add_row("📳 Getaran", f"{sensor.vibration:.3f} mm/s", str(vib_bar))
    
    layout["sensors"].update(Panel(sensor_table, title="[bold yellow]⚙️ LIVE DATA[/bold yellow]", border_style="green"))
    
    # Grafik tren suhu
    chart_str = sensor.get_chart()
    trend_panel = Panel(f"[bold green]Trend Suhu (°C) - 50 sample terakhir[/bold green]\n{chart_str}", 
                        title="📈 TREND ANALISIS", border_style="cyan")
    layout["trend"].update(trend_panel)
    
    # Panel status kanan
    status_text = f"""
[bold]🕒 Waktu Sistem:[/bold] {datetime.now().strftime('%H:%M:%S')}
[bold]🔄 Cycle Counter:[/bold] {counter}
[bold]📡 Modbus RTU:[/bold] [green]ACTIVE[/green]
[bold]🧠 PLC State:[/bold] [yellow]RUN[/yellow]
[bold]🔋 CPU Load:[/bold] {random.randint(15, 35)}%
    """
    layout["status"].update(Panel(status_text, title="[red] STATUS KONTROL [/red]", border_style="blue"))
    
    # Extra info (pesan inspiratif)
    extra_text = """
[bold magenta]▰ MOTO ENGINEER:[/bold magenta]
"Hardware itu mudah, software itu sulit,
integrasi adalah seni."

[bold]Terakhir update:[/bold] real-time
[bold]✨ Program keren by Raditya[/bold]
    """
    layout["extra"].update(Panel(extra_text, title="📢 QUOTE", border_style="green"))
    
    # Footer
    footer_text = "[dim]Tekan CTRL+C untuk keluar | Data simulasi untuk demo industri[/dim]"
    layout["footer"].update(Panel(footer_text, style="dim"))
    
    return layout

def main():
    sensor = SensorSim()
    counter = 0
    try:
        with Live(console=console, refresh_per_second=8, screen=True) as live:
            while True:
                sensor.update()
                counter += 1
                layout = create_layout(sensor, counter)
                live.update(layout)
                time.sleep(0.25)
    except KeyboardInterrupt:
        console.print("\n[bold red]>> Sistem dimatikan. Selamat bekerja! <<[/bold red]")
        time.sleep(0.5)
        os.system('cls' if os.name == 'nt' else 'clear')
        print("\n👋 Terima kasih telah menggunakan Industrial Dashboard - Raditya Rahmat\n")

if __name__ == "__main__":
    main()
