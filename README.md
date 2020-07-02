# pythongame

        import pygame
        import sys
        import random
        from time import sleep

        
        padWidth = 480 #화면 가로크기
        padHeight = 640 #화면 세로크기
        rockImage = ['rock01.png', 'rock02.png', 'rock03.png', 'rock04.png', 'rock05.png', 'rock06.png',\
                     'rock07.png', 'rock08.png', 'rock09.png', 'rock10.png', 'rock11.png', 'rock12.png',\
                     'rock13.png', 'rock14.png', 'rock15.png', 'rock16.png', 'rock17.png', 'rock18.png',\
                     'rock19.png', 'rock20.png', 'rock21.png', 'rock22.png', 'rock23.png', 'rock24.png',\
                     'rock25.png', 'rock26.png', 'rock27.png', 'rock28.png', 'rock29.png', 'rock30.png',]

        #운석을 맞춘 개수 계산
        def writeScore(count):
            global gamePad
            font = pygame.font.Font('NanumGothic.ttf', 20)
            text = font.render('파괴한 운석 수:' + str(count), True, (255, 255, 255))
            gamePad.blit(text,(10,0))

        #운석이 화면 아래로 통과한 개수
        def writePassed(count):
            global gamePad
            font = pygame.font.Font('NanumGothic.ttf', 20)
            text = font.render('놓친 운석:' + str(count), True, (255, 0, 0))
            gamePad.blit(text,(360,0))
        
        #게임 메시지 출력
        def writeMessage(text):
            global gamePad
            textfont = pygame.font.Font('NanumGothic.ttf', 80)
            text = textfont.render(text, True, (255, 0, 0))
            textpos = text.get_rect()
            textpos.center = (padWidth/2, padHeight/2)
            gamePad.blit(text, textpos)
            pygame.display.update()
            sleep(2)
            runGame()
        
        #전투기가 운석과 충돌했을 때 메시지 출력
        def crash():
            global gamePad
            writeMessage('전투기 파괴!')

        #게임오버 메시지 보이기
        def gameOver():
            global gamePad
            writeMessage('게임 오버!')
        def drawObject(obj, x, y):
            global gamepad
            gamePad.blit(obj, (x, y))
        
        def initGame():
            global gamePad, clock, background, fighter, missile, explosion
            pygame.init()
            gamePad = pygame.display.set_mode((padWidth, padHeight))
            pygame.display.set_caption('PyShooting') #게임이름
            background = pygame.image.load('background.png') #배경화면 로드
            fighter = pygame.image.load('fighter.png') #전투기그림
            missile = pygame.image.load('missile.png') #미사일그림
            explosion = pygame.image.load('explosion.png') #폭발그림
            clock = pygame.time.Clock()
        
        def runGame():
            global gamePad, clock, background, fighter, missile, explosion
        
            #전투기 크기
            fighterSize = fighter.get_rect().size
            fighterWidth = fighterSize[0]
            fighterHeight = fighterSize[1]
        
            #전투기 초기 위치 (x, y)
            x = padWidth * 0.45
            y = padHeight * 0.9
            fighterX = 0

            #무기 좌표 리스트
            missileXY = []
        
            #운석 랜덤 설정
            rock = pygame.image.load(random.choice(rockImage))
            rockSize = rock.get_rect().size #운석 크기
            rockWidth = rockSize[0]
            rockHeight = rockSize[1]
        
            #운석 초기 위치 설정
            rockX = random.randrange(0, padWidth - rockWidth)
            rockY = 0
            rockSpeed = 2
        
            #전투기 미사일에 운석이 맞았을 경우 True
            isShot = False
            shotCount = 0
            rockPassed = 0
            
            onGame = False
            while not onGame:
                for event in pygame.event.get():
                    if event.type in [pygame.QUIT]: #게임프로그램종료
                        pygame.quit()
                        sys.exit()
                    if event.type in [pygame.KEYDOWN]: #왼쪽
                        if event.key == pygame.K_LEFT:
                            fighterX -= 5
        
                        elif event.key == pygame.K_RIGHT: #오른쪽
                            fighterX += 5
        
                        elif event.key == pygame.K_SPACE: #발사
                            missileX = x + fighterWidth/2
                            missileY = y - fighterHeight
                            missileXY.append([missileX, missileY])
        
                    if event.type in [pygame.KEYUP]: #방향키떼면 전투기 스탑
                        if event.key == pygame.K_LEFT or event.key == pygame.K_RIGHT:
                            fighterX = 0
        
                drawObject(background, 0, 0) #배경화면 그리기
        
                #전투기 위치 재조정
                x += fighterX
                if x < 0:
                    x = 0
                elif x > padWidth - fighterWidth:
                    x = padWidth - fighterWidth
                
                #전투기가 운석과 충돌했는지 체크
                if y < rockY + rockHeight:
                    if(rockX > x and rockX < x + fighterWidth) or \
                                (rockX + rockWidth >  x and rockX + rockWidth < x + fighterWidth):
                                crash()
                
                        drawObject(fighter, x, y) #비행기를 게임화면 xy좌표에 그리기
                
                        #미사일 발사 그리기
                        if len(missileXY) != 0:
                            for i, bxy in enumerate(missileXY): #미사일 요소 반복
                                bxy[1] -= 10 #총알의 y좌표 -10 위로이동
                                missileXY[i][1] = bxy[1]
                
                                #미사일이 운석 맞췄을 때
                                if bxy[1] < rockY:
                                    if bxy[0] > rockX and bxy[0] < rockX + rockWidth:
                                        missileXY.remove(bxy)
                                        isShot = True
                                        shotCount += 1
                
                                if bxy[1] <= 0: #미사일 화면 밖으로 갈때
                                    try:
                                        missileXY.remove(bxy) #미사일 제거
                                    except:
                                        pass
                        if len(missileXY) != 0:
                            for bx, by in missileXY:
                                drawObject(missile, bx, by)
        
                        #운석 맞춘 점수 표시
                        writeScore(shotCount)
              
                        rockY += rockSpeed #운석 아래로 움직임
                
                        #운석이 지구로 떨어진 경우
                        if rockY > padHeight:
                            #랜덤 새 운석
                            rock = pygame.image.load(random.choice(rockImage))
                            rockSize = rock.get_rect().size
                            rockWidth = rockSize[0]
                            rockHeight = rockSize[1]
                            rockX = random.randrange(0, padWidth - rockWidth)
                            rockY = 0
                            rockPassed += 1
        
                if rockPassed == 3: #운석 3개 놓쳤을 때 게임오버
                    gameOver()

                #놓친 운석 수 표시
                writePassed(rockPassed)
        
                #운석을 맞춘 경우
                if isShot:
                    #운석폭발
                    drawObject(explosion, rockX, rockY) #운석 폭발 그림 생성
        
                    #새로운 랜덤 운석
                    rock = pygame.image.load(random.choice(rockImage))
                    rockSize = rock.get_rect().size
                    rockWidth = rockSize[0]
                    rockHeight = rockSize[1]
                    rockX = random.randrange(0, padWidth - rockWidth)
                    rockY = 0
                    isShot = False
                    
                    #운석 맞추면 속도 증가
                    rockSpeed += 0.2
                    if rockSpeed >= 10:
                        rockSpeed = 10
        
                drawObject(rock, rockX, rockY) #운석 그리기
                
                pygame.display.update() #게임화면 다시그림
        
                clock.tick(60) #게임 프레임설정
        
            pygame.quit() #파이게임종료
        
        initGame()
        runGame()
