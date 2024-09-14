using System;
using System.Drawing;
using System.Windows.Forms;

namespace PacManGame
{
    public partial class Form1 : Form
    {
        private PictureBox pacman;
        private PictureBox[] walls;
        private PictureBox[] enemies;
        private Timer gameTimer;
        private int pacmanSpeed = 5;
        private int enemySpeed = 2;
        private bool moveUp, moveDown, moveLeft, moveRight;
        private bool gameOver;

        private Random random = new Random();

        public Form1()
        {
            InitializeComponent();
            InitializeGame();
        }

        private void InitializeGame()
        {
            this.Width = 800;
            this.Height = 600;

            pacman = new PictureBox
            {
                Size = new Size(40, 40),
                Location = new Point(50, 50),
                BackColor = Color.Yellow
            };
            this.Controls.Add(pacman);

            walls = new PictureBox[10];
            for (int i = 0; i < walls.Length; i++)
            {
                walls[i] = new PictureBox
                {
                    Size = new Size(150, 20),
                    BackColor = Color.Blue
                };
                this.Controls.Add(walls[i]);
            }

            walls[0].Location = new Point(100, 100);
            walls[1].Location = new Point(300, 100);
            walls[2].Location = new Point(500, 100);
            walls[3].Location = new Point(100, 200);
            walls[4].Location = new Point(300, 200);
            walls[5].Location = new Point(500, 200);
            walls[6].Location = new Point(100, 300);
            walls[7].Location = new Point(300, 300);
            walls[8].Location = new Point(500, 300);
            walls[9].Location = new Point(300, 400);

            enemies = new PictureBox[2];
            for (int i = 0; i < enemies.Length; i++)
            {
                enemies[i] = new PictureBox
                {
                    Size = new Size(40, 40),
                    BackColor = Color.Red,
                    Location = new Point(200 + (i * 100), 50)
                };
                this.Controls.Add(enemies[i]);
            }

            gameTimer = new Timer
            {
                Interval = 20
            };
            gameTimer.Tick += UpdateGame;
            gameTimer.Start();

            this.KeyDown += new KeyEventHandler(OnKeyDown);
            this.KeyUp += new KeyEventHandler(OnKeyUp);

            gameOver = false;
        }

        private void UpdateGame(object sender, EventArgs e)
        {
            if (!gameOver)
            {
                MovePacman();
                MoveEnemies();
                CheckCollisions();
            }
        }

        private void MovePacman()
        {
            int newX = pacman.Left;
            int newY = pacman.Top;

            if (moveUp) newY -= pacmanSpeed;
            if (moveDown) newY += pacmanSpeed;
            if (moveLeft) newX -= pacmanSpeed;
            if (moveRight) newX += pacmanSpeed;

            Rectangle futurePacman = new Rectangle(newX, newY, pacman.Width, pacman.Height);

            bool collisionDetected = false;
            foreach (var wall in walls)
            {
                if (futurePacman.IntersectsWith(wall.Bounds))
                {
                    collisionDetected = true;
                    break;
                }
            }

            if (!collisionDetected)
            {
                pacman.Left = newX;
                pacman.Top = newY;
            }
        }

        private void MoveEnemies()
        {
            foreach (var enemy in enemies)
            {
                int enemyX = enemy.Left;
                int enemyY = enemy.Top;

                // Determinar la dirección del movimiento basado en la posición de Pac-Man
                if (Math.Abs(pacman.Top - enemy.Top) > Math.Abs(pacman.Left - enemy.Left))
                {
                    // Mover en dirección vertical si la distancia es mayor
                    if (enemy.Top < pacman.Top)
                        enemyY += enemySpeed;
                    else
                        enemyY -= enemySpeed;
                }
                else
                {
                    // Mover en dirección horizontal
                    if (enemy.Left < pacman.Left)
                        enemyX += enemySpeed;
                    else
                        enemyX -= enemySpeed;
                }

                // Crear un rectángulo que represente la posición futura del enemigo
                Rectangle futureEnemy = new Rectangle(enemyX, enemyY, enemy.Width, enemy.Height);

                // Verificar si el nuevo movimiento causa una colisión con las paredes
                bool collisionDetected = false;
                foreach (var wall in walls)
                {
                    if (futureEnemy.IntersectsWith(wall.Bounds))
                    {
                        collisionDetected = true;
                        break;
                    }
                }

                // Si hay colisión, cambiar de dirección
                if (collisionDetected)
                {
                    if (Math.Abs(pacman.Left - enemy.Left) > Math.Abs(pacman.Top - enemy.Top))
                    {
                        // Cambiar a movimiento vertical
                        if (enemy.Top < pacman.Top)
                            enemyY += enemySpeed;
                        else
                            enemyY -= enemySpeed;
                    }
                    else
                    {
                        // Cambiar a movimiento horizontal
                        if (enemy.Left < pacman.Left)
                            enemyX += enemySpeed;
                        else
                            enemyX -= enemySpeed;
                    }
                }

                enemy.Left = enemyX;
                enemy.Top = enemyY;
            }
        }

        private void CheckCollisions()
        {
            foreach (var wall in walls)
            {
                if (pacman.Bounds.IntersectsWith(wall.Bounds))
                {
                    StopMovement();
                }
            }

            foreach (var enemy in enemies)
            {
                if (pacman.Bounds.IntersectsWith(enemy.Bounds) && !gameOver)
                {
                    gameTimer.Stop();
                    gameOver = true;
                    MessageBox.Show("¡Perdiste! El enemigo te atrapó.");
                }
            }
        }

        private void StopMovement()
        {
            moveUp = moveDown = moveLeft = moveRight = false;
        }

        private void OnKeyDown(object sender, KeyEventArgs e)
        {
            if (e.KeyCode == Keys.Up) moveUp = true;
            if (e.KeyCode == Keys.Down) moveDown = true;
            if (e.KeyCode == Keys.Left) moveLeft = true;
            if (e.KeyCode == Keys.Right) moveRight = true;
        }

        private void OnKeyUp(object sender, KeyEventArgs e)
        {
            if (e.KeyCode == Keys.Up) moveUp = false;
            if (e.KeyCode == Keys.Down) moveDown = false;
            if (e.KeyCode == Keys.Left) moveLeft = false;
            if (e.KeyCode == Keys.Right) moveRight = false;
        }
    }
}


