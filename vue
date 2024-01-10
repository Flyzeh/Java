package vue;

import java.awt.BorderLayout;
import java.awt.Color;
import java.awt.Dimension;
import java.awt.Graphics;
import java.awt.Image;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.awt.event.KeyEvent;
import java.awt.event.KeyListener;
import java.awt.event.MouseEvent;
import java.awt.event.MouseListener;
import java.awt.event.MouseMotionListener;
import javax.swing.JFrame;
import javax.swing.JLabel;
import javax.swing.JOptionPane;
import javax.swing.JPanel;
import modele.Aventurier;
import modele.Labyrinthe;
import modele.Case;
import modele.Ocean;
import javax.swing.*;
import java.awt.event.KeyEvent;
import java.awt.event.KeyListener;


/**
 * Fenêtre de visualisation du labyrinthe.
 */
public class Vue extends JFrame
{
	/*------------*/
	/* Propriétés */
	/*------------*/

	/**
	 * Référence vers le labyrinthe que la classe Vue va visualiser.
	 */
	private final Labyrinthe labyrinthe;

	/*----- Barre d'état de la fenêtre -----*/
	private final JLabel barre_etat;

	/*----- Zone de dessin -----*/
	private final Dessin dessin;


	/*--------------*/
	/* Constructeur */
	/*--------------*/

	public Vue (int x, int y, Labyrinthe labyrinthe)
		{
		/*----- Lien avec le labyrinthe -----*/
		this.labyrinthe = labyrinthe;

		/*----- Paramètres de la fenêtre -----*/
		this.setTitle("Labyrinthe");
		this.setResizable(false);
		this.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
		this.setLocation(x,y);
		this.setLayout(new BorderLayout());

		/*----- Zone de dessin -----*/
		this.dessin = new Dessin(600);
		this.dessin.setFocusable(true);
		this.dessin.startGameThread();

		/*----- Attachement des écouteurs des évènements souris et clavier -----*/
		this.dessin.addMouseListener(this.dessin);
		this.dessin.addMouseMotionListener(this.dessin);
		this.dessin.addKeyListener(this.dessin);
		this.add(this.dessin, BorderLayout.CENTER);

		/*----- Barre d'état de la fenêtre -----*/
		this.barre_etat = new JLabel("Barre d'état");
		this.add(this.barre_etat, BorderLayout.SOUTH);

		/*----- Pour ajuster la fenêtre à son contenu et la rendre visible -----*/
		this.pack();
		this.setVisible(true);
		
		String nomJoueur = JOptionPane.showInputDialog(this, "Entrez votre nom :");
	    
	    // Vérifier si le joueur a annulé la saisie ou n'a rien entré
	    if (nomJoueur == null || nomJoueur.trim().isEmpty()) {
	        // Traiter l'annulation ou l'entrée vide, par exemple, quitter le jeu
	        System.exit(0);
	    } else {
	        // Utiliser le nom du joueur comme vous le souhaitez, par exemple, l'afficher dans la barre d'état
	        barre_etat.setText("Nom du joueur : " + nomJoueur);
	    }

		}


	/*----------------*/
	/* Classe interne */
	/*----------------*/

	class Dessin extends JPanel implements KeyListener, MouseListener, MouseMotionListener, Runnable
		{
		/*----- Propriétés de la classe interne -----*/
		int largeur;
		int taille_case;
		
		ImageIcon[] personnageSprites = new ImageIcon[8];

		/*----- Constructeur de la classe interne -----*/
		public Dessin (int larg)
			{
			/*----- Initialisation des données -----*/
			this.taille_case = larg / labyrinthe.getTaille();
			this.largeur = this.taille_case * labyrinthe.getTaille();

			/*----- Paramètre du JPanel -----*/
			this.setPreferredSize(new Dimension(this.largeur, this.largeur));
			for (int i = 0; i < 8; i++) {
		        personnageSprites[i] = new ImageIcon("C:\\Users\\giani\\Downloads\\ProjetLabyrinthe\\ProjetLabyrintheEtd\\data\\Perso\\personnage" + (i + 1) + ".png");
		    }
			
		
		}
		int spriteCounter = 0;
		int spriteNumber = 1;
		int FPS = 60;
		Thread gameThread;

		public void startGameThread() {
			gameThread = new Thread(this);
			gameThread.start(); // permet d'activer la méthode run
		}
		
		@Override
		public void run() {
			// TODO Auto-generated method stub
			double drawInterval = 1000000000 / FPS;
		    double nextDrawTime = System.nanoTime() + drawInterval;

		    while (gameThread != null) {
		        update();
		        
		        repaint();

		        try {
		            double remainingTime = nextDrawTime - System.nanoTime();
		            remainingTime = remainingTime / 1000000;

		            if (remainingTime < 0) {
		                remainingTime = 0;
		            }
		            Thread.sleep((long) remainingTime);

		            nextDrawTime += drawInterval;
		        } catch (InterruptedException e) {
		            e.printStackTrace();
		        }
		    }
		}
		
		public void update() {
		    spriteCounter++;
		    if (spriteCounter > 12) {
		        if (spriteNumber == 1) {
		            spriteNumber = 2;
		        } else if (spriteNumber == 2) {
		            spriteNumber = 1;
		        }
		        spriteCounter = 0; // Réinitialiser le compteur après avoir changé le sprite
		    }
		}
		
		/**
		 * Méthode qui permet de dessiner ou redessinner le labyrinthe lorsque
		 * la méthode repaint() est appelée sur la classe Dessin.
		 */
		@Override
		public void paint (Graphics g)
			{
			/*----- On efface le dessin en entier -----*/
			g.setColor(Color.LIGHT_GRAY);
			g.fillRect(0,0,this.largeur,this.largeur);

			/*----- Affichage des cases du labyrinthe -----*/
			for (int i=0; i<labyrinthe.getTaille(); i++)
				for (int j=0; j<labyrinthe.getTaille(); j++)
					{
					if (labyrinthe.getCase(i,j).getClassName().equals("Mur")) {
						 ImageIcon murIcon = new ImageIcon("C:\\Users\\giani\\Downloads\\ProjetLabyrinthe\\ProjetLabyrintheEtd\\data\\mur.gif");
						 Image imageMur = murIcon.getImage();
						 g.drawImage(imageMur, taille_case * j, taille_case * i, taille_case, taille_case, this); 
					}
					if (labyrinthe.getCase(i,j).getClassName().equals("Espace")) {
							 ImageIcon CheminIcon = new ImageIcon("C:\\Users\\giani\\Downloads\\ProjetLabyrinthe\\ProjetLabyrintheEtd\\data\\floor01.png");
							 Image imageChemin = CheminIcon.getImage();
							 g.drawImage(imageChemin, taille_case * j, taille_case * i, taille_case, taille_case, this);
					}
					if (labyrinthe.getCase(i, j).getClassName().equals("Ocean")) {
						 ImageIcon OceanIcon = new ImageIcon("C:\\Users\\giani\\Downloads\\ProjetLabyrinthe\\ProjetLabyrintheEtd\\data\\water01.png");
						 Image imageOcean = OceanIcon.getImage();
						 g.drawImage(imageOcean, taille_case * j, taille_case * i, taille_case, taille_case, this);
					}
					if (labyrinthe.getCase(i, j).getClassName().equals("Desert"))	g.setColor(Color.YELLOW);
					if (labyrinthe.getCase(i, j).getClassName().equals("Fee"))		g.setColor(Color.PINK);
					if (labyrinthe.getCase(i, j).getClassName().equals("Jungle"))	g.setColor(Color.GREEN);
					if (labyrinthe.getCase(i, j).getClassName().equals("Taiga"))	g.setColor(Color.DARK_GRAY);
					if (!labyrinthe.getCase(i,j).getClassName().equals("Mur") && !labyrinthe.getCase(i, j).getClassName().equals("Ocean") && !labyrinthe.getCase(i,j).getClassName().equals("Espace"))
						/*----- Affichage de la case sous forme d'un rectangle plein -----*/
						g.fillRect(taille_case*j, taille_case*i, taille_case, taille_case);
					}

			/*----- Affichage de l'aventurier -----*/
			Aventurier aventurier = labyrinthe.getAventurier();
			int direction = labyrinthe.getAventurier().getDirection();
			if (direction == 0 || direction == 1) {
			if (spriteNumber == 1)
				labyrinthe.getAventurier().setDirection(0); 
			if (spriteNumber == 2)
				labyrinthe.getAventurier().setDirection(1); 
			}
			if (direction == 2 || direction == 3) {
				if (spriteNumber == 1)
					labyrinthe.getAventurier().setDirection(2); 
				if (spriteNumber == 2)
					labyrinthe.getAventurier().setDirection(3); 
			}
			if (direction == 4 || direction == 5) {
				if (spriteNumber == 1)
					labyrinthe.getAventurier().setDirection(4); 
				if (spriteNumber == 2)
					labyrinthe.getAventurier().setDirection(5); 
			}
			if (direction == 6 || direction == 7) {
				if (spriteNumber == 1)
					labyrinthe.getAventurier().setDirection(6); 
				if (spriteNumber == 2)
					labyrinthe.getAventurier().setDirection(7); 
			}
			// Redimensionner l'image si nécessaire
	        Image image = personnageSprites[direction].getImage();
	        int nouvelleLargeur = (int) (taille_case / 2 * 1.5);
	        int nouvelleHauteur = (int) (taille_case / 2 * 1.5);
	        Image nouvelleImage = image.getScaledInstance(taille_case / 2, taille_case / 2, Image.SCALE_SMOOTH);
	        personnageSprites[direction] = new ImageIcon(nouvelleImage);

	        // Afficher le personnage avec le bon sprite
	        g.drawImage(personnageSprites[direction].getImage(),
	                taille_case * aventurier.getY() + taille_case / 4,
	                taille_case * aventurier.getX() + taille_case / 4,
	                nouvelleLargeur,
	                nouvelleHauteur,
	                this);

	        
			}
		/**
		 * Gestion des interactions souris et clavier sur le labyrinthe.
		 */
		@Override
		public void mouseClicked (MouseEvent e)
			{
			/*----- Lecture de la position de la souris lors du clic sur l'objet Dessin -----*/
			int x = e.getX();
			int y = e.getY();
			
			/*----- Recherche des coordonnées de la case du labyrinthe sur laquelle le joueur a cliqué -----*/
			int ligne = y / this.taille_case;
			int colonne = x / this.taille_case;

			/*----- On regarde si l'aventurier est sur la case sur laquelle on vient de cliquer -----*/
			String msgAventurier = "";
			if (labyrinthe.getAventurier().getX() == ligne && labyrinthe.getAventurier().getY() == colonne)
				msgAventurier = "\nL'aventurier est sur cette case.";

			/*----- Etat de la case -----*/
			JOptionPane.showMessageDialog(this, "Vous avez cliqué sur la case (" + ligne + "," + colonne +").\nC'est un "
												+ labyrinthe.getCase(ligne, colonne).getClassName()
												+ "." + msgAventurier);
			
			repaint();
			}

		@Override
		public void mousePressed (MouseEvent e) { }

		@Override
		public void mouseReleased (MouseEvent e) { }

		@Override
		public void mouseEntered (MouseEvent e) { }

		@Override
		public void mouseExited (MouseEvent e) { }

		@Override
		public void mouseDragged (MouseEvent e) { }

		@Override
		public void mouseMoved (MouseEvent e)
			{
			barre_etat.setText(" Position de la souris : " + e.getX() + " " + e.getY());
			}

		@Override
		public void keyTyped (KeyEvent e) { }

		@Override
		public void keyPressed (KeyEvent e) { }

		@Override
		public void keyReleased (KeyEvent e)
			{
			/**
			 * Déplacement de l'aventurier dans le labyrinthe.
			 */
			int pos;
			if (e.getKeyCode() == KeyEvent.VK_DOWN) {
				if (spriteNumber == 1)
					labyrinthe.getAventurier().setDirection(0); 
				if (spriteNumber == 2)
					labyrinthe.getAventurier().setDirection(1); 
		    } else if (e.getKeyCode() == KeyEvent.VK_UP) {
		    	if (spriteNumber == 1)
		    		labyrinthe.getAventurier().setDirection(6); 
		    	if (spriteNumber == 2)
					labyrinthe.getAventurier().setDirection(7); 
		    } else if (e.getKeyCode() == KeyEvent.VK_RIGHT) {
		    	if (spriteNumber == 1)
		    		labyrinthe.getAventurier().setDirection(4); 
		    	if (spriteNumber == 2)
					labyrinthe.getAventurier().setDirection(5); 
		    } else if (e.getKeyCode() == KeyEvent.VK_LEFT) {
		    	if (spriteNumber == 1)
		    		labyrinthe.getAventurier().setDirection(2); 
		    	if (spriteNumber == 2)
					labyrinthe.getAventurier().setDirection(3); 
		    }
			
			/*----- Vers le bas -----*/
			if (e.getKeyCode() == KeyEvent.VK_DOWN)
				{
				pos = labyrinthe.getAventurier().getX();
				
				if (!labyrinthe.getCase(pos + 1, labyrinthe.getAventurier().getY()).getClassName().equals("Mur")) {
				    Case caseSuivante = labyrinthe.getCase(pos + 1, labyrinthe.getAventurier().getY());

				    // Vérifier si la case est un océan et si le joueur ne veut pas utiliser un bateau
				    if (caseSuivante.getClassName().equals("Ocean") && !Ocean.UtiliserBateau()) {
				        System.out.println("Tu ne peux pas aller sur une case d'océan sans un bateau !");
				    } else {
				        // Si la case n'est pas un mur ou un océan sans bateau, déplacer l'aventurier
				        labyrinthe.getAventurier().setX(pos + 1);
				    }
				}

				}
				

			/*----- Vers le haut -----*/
			if (e.getKeyCode() == KeyEvent.VK_UP)
				{
				pos = labyrinthe.getAventurier().getX();
				if (!labyrinthe.getCase(pos - 1, labyrinthe.getAventurier().getY()).getClassName().equals("Mur")) {
				    Case caseSuivante = labyrinthe.getCase(pos - 1, labyrinthe.getAventurier().getY());

				    // Vérifier si la case est un océan et si le joueur ne veut pas utiliser un bateau
				    if (caseSuivante.getClassName().equals("Ocean") && !Ocean.UtiliserBateau()) {
				        System.out.println("Tu ne peux pas aller sur une case d'océan sans un bateau !");
				    } else {
				        // Si la case n'est pas un mur ou un océan sans bateau, déplacer l'aventurier
				        labyrinthe.getAventurier().setX(pos - 1);
				    }
				}
				}

			/*----- Vers la droite -----*/
			if (e.getKeyCode() == KeyEvent.VK_RIGHT) {
				pos = labyrinthe.getAventurier().getY();
				Case caseSuivante = labyrinthe.getCase(labyrinthe.getAventurier().getX(), pos + 1);

	        // Vérifier si la case n'est pas un mur
	        if (!caseSuivante.getClassName().equals("Mur")) {
	            // Vérifier si la case est un océan et si le joueur ne veut pas utiliser un bateau
	            if (caseSuivante.getClassName().equals("Ocean") && !Ocean.UtiliserBateau()) {
	                System.out.println("Tu ne peux pas aller sur une case d'océan sans un bateau !");
	            } else {
	                // Si la case n'est pas un mur ou un océan sans bateau, déplacer l'aventurier
	                labyrinthe.getAventurier().setY(pos + 1);
	            }
	        }
	    }


			/*----- Vers la gauche -----*/
			if (e.getKeyCode() == KeyEvent.VK_LEFT)
				{
				pos = labyrinthe.getAventurier().getY();
				Case caseSuivante = labyrinthe.getCase(labyrinthe.getAventurier().getX(), pos - 1);

	        // Vérifier si la case n'est pas un mur
	        if (!caseSuivante.getClassName().equals("Mur")) {
	            // Vérifier si la case est un océan et si le joueur ne veut pas utiliser un bateau
	            if (caseSuivante.getClassName().equals("Ocean") && !Ocean.UtiliserBateau()) {
	                System.out.println("Tu ne peux pas aller sur une case d'océan sans un bateau !");
	            } else {
	                // Si la case n'est pas un mur ou un océan sans bateau, déplacer l'aventurier
	                labyrinthe.getAventurier().setY(pos - 1);
	            }
	        }
	    }
			
			/*----- On déclenche l'action de la case sur laquelle est l'aventurier -----*/
			labyrinthe.action();

			/*----- On refait le dessin -----*/
			repaint();
			}

		

		} /*----- Fin de la classe interne Dessin -----*/

} /*----- Fin de la classe Vue -----*/
