## Same in Python as coding challenge, but it does not work prop correct


import os
import math
import time
import keyboard
from ctypes import windll, create_unicode_buffer, c_int, byref, Structure, c_short

# Definition der COORD-Struktur
class COORD(Structure):
    _fields_ = [("X", c_short), ("Y", c_short)]

# Konsolenabmessungen
nScreenWidth = 120  # Konsolenbildschirmbreite (Spalten)
nScreenHeight = 40  # Konsolenbildschirmhöhe (Zeilen)
nMapWidth = 16      # Weltkartenbreite
nMapHeight = 16     # Weltkartenhöhe

# Startposition und Eigenschaften des Spielers
fPlayerX = 14.7
fPlayerY = 5.09
fPlayerA = 0.0
fFOV = math.pi / 4.0  # Sichtfeld
fDepth = 16.0         # Maximale Rendering-Entfernung
fSpeed = 5.0          # Spielergeschwindigkeit

# Weltkarte Definition
map = """
#########.......
#...............
#.......########
#..............#
#......##......#
#......##......#
#..............#
###............#
##.............#
#......####..###
#......#.......#
#......#.......#
#..............#
#......#########
#..............#
################
"""

# Bildschirmpuffer initialisieren
screen = create_unicode_buffer(nScreenWidth * nScreenHeight)
hConsole = windll.kernel32.GetStdHandle(-11)  # Handle für die Konsolenausgabe abrufen

# Überprüfen, ob das Konsolenhandle korrekt ist
if hConsole == 0 or hConsole == -1:
    raise Exception("Konsolenhandle konnte nicht abgerufen werden.")

windll.kernel32.SetConsoleActiveScreenBuffer(hConsole)

# Zeitverfolgung
tp1 = time.time()

while True:
    # Zeitdifferential für Frame
    tp2 = time.time()
    fElapsedTime = tp2 - tp1
    tp1 = tp2

    # Vermeidung von Division durch Null
    if fElapsedTime == 0.0:
        fElapsedTime = 0.0001

    # CCW-Rotation handhaben
    if keyboard.is_pressed('a'):
        fPlayerA -= fSpeed * 0.75 * fElapsedTime

    # CW-Rotation handhaben
    if keyboard.is_pressed('d'):
        fPlayerA += fSpeed * 0.75 * fElapsedTime

    # Vorwärtsbewegung und Kollision handhaben
    if keyboard.is_pressed('w'):
        fPlayerX += math.sin(fPlayerA) * fSpeed * fElapsedTime
        fPlayerY += math.cos(fPlayerA) * fSpeed * fElapsedTime
        if map[int(fPlayerY) * nMapWidth + int(fPlayerX)] == '#':
            fPlayerX -= math.sin(fPlayerA) * fSpeed * fElapsedTime
            fPlayerY -= math.cos(fPlayerA) * fSpeed * fElapsedTime

    # Rückwärtsbewegung und Kollision handhaben
    if keyboard.is_pressed('s'):
        fPlayerX -= math.sin(fPlayerA) * fSpeed * fElapsedTime
        fPlayerY -= math.cos(fPlayerA) * fSpeed * fElapsedTime
        if map[int(fPlayerY) * nMapWidth + int(fPlayerX)] == '#':
            fPlayerX += math.sin(fPlayerA) * fSpeed * fElapsedTime
            fPlayerY += math.cos(fPlayerA) * fSpeed * fElapsedTime

    for x in range(nScreenWidth):
        # Berechne den projizierten Strahlwinkel in den Welt-Raum
        fRayAngle = (fPlayerA - fFOV / 2.0) + (x / nScreenWidth) * fFOV

        fDistanceToWall = 0
        bHitWall = False
        bBoundary = False

        fEyeX = math.sin(fRayAngle)
        fEyeY = math.cos(fRayAngle)

        # Strahl schrittweise vom Spieler aus senden
        while not bHitWall and fDistanceToWall < fDepth:
            fDistanceToWall += 0.1

            nTestX = int(fPlayerX + fEyeX * fDistanceToWall)
            nTestY = int(fPlayerY + fEyeY * fDistanceToWall)

            # Prüfe, ob außerhalb der Grenzen
            if nTestX < 0 or nTestX >= nMapWidth or nTestY < 0 or nTestY >= nMapHeight:
                bHitWall = True
                fDistanceToWall = fDepth
            else:
                if map[nTestY * nMapWidth + nTestX] == '#':
                    bHitWall = True

        # Berechne Abstand zur Decke und zum Boden
        nCeiling = int(nScreenHeight / 2.0 - nScreenHeight / fDistanceToWall)
        nFloor = nScreenHeight - nCeiling

        # Wände basierend auf der Entfernung schattieren
        nShade = ' '
        if fDistanceToWall <= fDepth / 4.0:
            nShade = '█'  # Sehr nah
        elif fDistanceToWall < fDepth / 3.0:
            nShade = '▓'
        elif fDistanceToWall < fDepth / 2.0:
            nShade = '▒'
        elif fDistanceToWall < fDepth:
            nShade = '░'
        else:
            nShade = ' '  # Zu weit entfernt

        if bBoundary:
            nShade = ' '  # Abdunkeln

        for y in range(nScreenHeight):
            if y <= nCeiling:
                screen[y * nScreenWidth + x] = ' '
            elif y > nCeiling and y <= nFloor:
                screen[y * nScreenWidth + x] = nShade
            else:
                b = 1.0 - (y - nScreenHeight / 2.0) / (nScreenHeight / 2.0)
                if b < 0.25:
                    nShade = '#'
                elif b < 0.5:
                    nShade = 'x'
                elif b < 0.75:
                    nShade = '.'
                elif b < 0.9:
                    nShade = '-'
                else:
                    nShade = ' '
                screen[y * nScreenWidth + x] = nShade

    # Statistiken anzeigen
    stats = f"X={fPlayerX:.2f}, Y={fPlayerY:.2f}, A={fPlayerA:.2f}, FPS={1.0 / fElapsedTime:.2f}"
    for i, char in enumerate(stats):
        screen[i] = char

    # Karte anzeigen
    for nx in range(nMapWidth):
        for ny in range(nMapWidth):
            screen[(ny + 1) * nScreenWidth + nx] = map[ny * nMapWidth + nx]

    screen[int(fPlayerY + 1) * nScreenWidth + int(fPlayerX)] = 'P'

    # Schreibe den Konsolenpuffer auf den Bildschirm
    written = c_int(0)
    coord = COORD(0, 0)  # Verwende COORD statt eines Tupels
    windll.kernel32.WriteConsoleOutputCharacterW(hConsole, screen, nScreenWidth * nScreenHeight, coord, byref(written))

    time.sleep(0.01)
