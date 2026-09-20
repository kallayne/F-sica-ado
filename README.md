# F-sica-ado
import numpy as np
import matplotlib.pyplot as plt
from matplotlib.widgets import Slider, Button
from matplotlib.animation import FuncAnimation

# ==========================================
# 1. FUNÇÕES FÍSICAS E CÁLCULO ANALÍTICO
# ==========================================
def calcular_resultados(v0, angulo_graus, y0, g):
    """
    Calcula os resultados analíticos do lançamento de projétil sem resistência do ar.
    Retorna o tempo de voo, a altura máxima e o alcance horizontal.
    """
    theta = np.radians(angulo_graus)
    
    # Tempo de voo
    t_voo = (v0 * np.sin(theta) + np.sqrt((v0 * np.sin(theta))**2 + 2 * g * y0)) / g
    
    # Altura máxima
    y_max = y0 + ((v0 * np.sin(theta))**2) / (2 * g)
    
    # Alcance horizontal total
    R = v0 * np.cos(theta) * t_voo
    
    return t_voo, y_max, R

def calcular_trajetoria(v0, angulo_graus, y0, g, t_voo):
    """
    Gera as coordenadas x e y da trajetória do projétil baseadas no tempo.
    """
    theta = np.radians(angulo_graus)
    t = np.linspace(0, t_voo, 200) # Cria 200 pontos de tempo
    
    x = v0 * np.cos(theta) * t
    y = y0 + v0 * np.sin(theta) * t - 0.5 * g * t**2
    
    return x, y

# ==========================================
# 2. CONFIGURAÇÃO DA INTERFACE (GUI)
# ==========================================
# Configuração da janela principal e área do gráfico
fig, ax = plt.subplots(figsize=(10, 7))
plt.subplots_adjust(left=0.1, bottom=0.35, top=0.85)

# Elementos gráficos da trajetória
linha_trajetoria, = ax.plot([], [], 'b-', lw=2, label='Trajetória')
ponto_projetil, = ax.plot([], [], 'ro', markersize=8, label='Projétil')

# Configurações dos eixos do gráfico
ax.set_xlabel('Distância Horizontal x (m)')
ax.set_ylabel('Altura y (m)')
ax.set_title('Simulador Interativo de Lançamento de Projéteis')
ax.grid(True)

# Caixa de texto para exibir os resultados numéricos
texto_resultados = plt.figtext(0.15, 0.9, '', fontsize=10, bbox=dict(facecolor='white', alpha=0.8))

# Definição das posições e criação dos controles deslizantes (Sliders)
axcolor = 'lightgoldenrodyellow'
ax_v0 = plt.axes([0.15, 0.25, 0.65, 0.03], facecolor=axcolor)
ax_ang = plt.axes([0.15, 0.20, 0.65, 0.03], facecolor=axcolor)
ax_y0 = plt.axes([0.15, 0.15, 0.65, 0.03], facecolor=axcolor)
ax_g = plt.axes([0.15, 0.10, 0.65, 0.03], facecolor=axcolor)

sl_v0 = Slider(ax_v0, 'Veloc. Inicial (m/s)', 5.0, 150.0, valinit=50.0)
sl_ang = Slider(ax_ang, 'Ângulo (°)', 1.0, 89.0, valinit=45.0)
sl_y0 = Slider(ax_y0, 'Altura Inicial (m)', 0.0, 50.0, valinit=10.0)
sl_g = Slider(ax_g, 'Gravidade (m/s²)', 1.6, 24.8, valinit=9.8)

# Variáveis globais para a animação
anim = None
x_data, y_data = [], []

# ==========================================
# 3. LÓGICA DE ATUALIZAÇÃO E EVENTOS
# ==========================================
def atualizar_grafico(val):
    """
    Recalcula a física e atualiza o gráfico sempre que um slider for movido.
    """
    global x_data, y_data
    
    # Obtém os valores atuais dos controles
    v0, ang, y0, g = sl_v0.val, sl_ang.val, sl_y0.val, sl_g.val
    
    # Calcula os resultados e a trajetória
    t_voo, y_max, R = calcular_resultados(v0, ang, y0, g)
    x_data, y_data = calcular_trajetoria(v0, ang, y0, g, t_voo)
    
    # Atualiza a linha do gráfico
    linha_trajetoria.set_data(x_data, y_data)
    
    # Ajusta os limites dos eixos para manter a proporção da curva visível
    ax.set_xlim(0, max(R * 1.1, 10))
    ax.set_ylim(0, max(y_max * 1.1, 10))
    
    # Reseta a posição inicial do projétil
    ponto_projetil.set_data([x_data[0]], [y_data[0]]) 
    
    # Atualiza o painel de resultados numéricos
    texto_resultados.set_text(
        f"Resultados:\nAlcance (R): {R:.2f} m\nAltura Máx: {y_max:.2f} m\nTempo de Voo: {t_voo:.2f} s"
    )
    
    # Redesenha a tela
    fig.canvas.draw_idle()

# Associa os sliders à função de atualização
sl_v0.on_changed(atualizar_grafico)
sl_ang.on_changed(atualizar_grafico)
sl_y0.on_changed(atualizar_grafico)
sl_g.on_changed(atualizar_grafico)

# ==========================================
# 4. BOTÃO DE ANIMAÇÃO
# ==========================================
# Criação do botão de disparo
ax_btn = plt.axes([0.85, 0.02, 0.1, 0.05])
btn_lancar = Button(ax_btn, 'Lançar', color='lightblue')

def animar_frame(i):
    """Atualiza a posição do projétil para o frame atual da animação."""
    if i < len(x_data):
        ponto_projetil.set_data([x_data[i]], [y_data[i]])
    return ponto_projetil,

def disparar_animacao(event):
    """
    Inicia o deslocamento do projétil pela trajetória calculada.
    """
    global anim
    
    # Se uma animação já estiver rodando, para ela primeiro
    if anim:
        anim.event_source.stop()
        
    # Garante que os dados mais recentes estejam calculados
    atualizar_grafico(None) 
    
    # Inicia a animação percorrendo os pontos x_data e y_data
    anim = FuncAnimation(fig, animar_frame, frames=len(x_data), interval=20, blit=True, repeat=False)
    fig.canvas.draw()

# Associa o botão à função de disparo
btn_lancar.on_clicked(disparar_animacao)

# Força a primeira atualização do gráfico ao abrir o programa
atualizar_grafico(None)

# Mostra a janela com o programa
plt.show()
