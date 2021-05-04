# GameOfLifeProject
This is my Implementation of Game of Life, a JetBrains Academy project Stage 5 / 5. About:
"Get firsthand experience of creating a small inhabited universe and observe the many patterns in which this “life” can evolve. Generation by generation, watch the cells come and go, reacting to their environment, perishing from loneliness or finding comfort in company. In this project, you will write a simple “Game of Life”, a classic toy for programmers to entertain and educate themselves. Careful: might be hypnotizing!" 

Frame below depicts how program works:


![image](https://user-images.githubusercontent.com/69851038/117053682-a0363380-acef-11eb-9fc0-342856348073.png)
--------------------------------------------------------------
My code:

import java.awt.BorderLayout;

import java.util.Queue;

import javax.swing.JFrame;

public class GameOfLife extends JFrame {

    static ControlPanel controlPanel;
    static GenerationPanel generationPanel;
    static GameOfLifeAlgo gl = new GameOfLifeAlgo();
    static int genNum = 10000;
    static int universeSize = 100;
    static int genCounter = 0;

    public GameOfLife() throws InterruptedException {
        super("Game of life");
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setSize(370, 300);
        setLocationRelativeTo(null);

        setLayout(new BorderLayout());

        controlPanel = new ControlPanel();

        add(BorderLayout.WEST, controlPanel);



        generationPanel = new GenerationPanel(universeSize, genNum);
        add(BorderLayout.CENTER, generationPanel);

        setVisible(true);
    }

    public static void main(String[] args) throws InterruptedException { 
        new GameOfLife();
        runIt();
    }


    static void runIt() throws InterruptedException {
        gl = new GameOfLifeAlgo();
        Queue <boolean[][]> gens = gl.generationsCreator(universeSize, genNum);


        while(gens.size() > 0) {
            if (!ControlPanel.paused) {
                boolean[][] aux = gens.poll();
                refresh(aux, genCounter, gl.getAlive(aux));
                genCounter++;
            } else {
                Thread.onSpinWait();
            }
            if(ControlPanel.reseted) {
                gens.clear();
                genCounter = 0;
                gens = gl.generationsCreator(universeSize, genNum);
                ControlPanel.reseted = false;
            }
        }
        if (gens.isEmpty()) {
            System.exit(0);
        }
    }


    public static void refresh(boolean[][] universe, int genCounter,  int alive) throws InterruptedException {
        ControlPanel.generationLabel.setText("Generation #" + genCounter);
        ControlPanel.aliveLabel.setText("Alive: " + alive);


        for (int i = 0; i < universe.length; i++) {
            for (int j = 0; j < universe.length; j++) {
                GenerationPanel.labels[i][j].setOpaque(universe[i][j]);
            }
        }

        controlPanel.repaint();
        generationPanel.repaint();

        Thread.sleep(ControlPanel.evolutionSpeed);
    }


}

import java.util.LinkedList;

import java.util.Queue;

import java.util.Random;

class GameOfLifeAlgo {

    public int getAlive(boolean[][] generation) {
        int alive = 0;
        for (boolean[] elements : generation) {
            for (boolean element : elements) {
                if (element) alive++;
            }
        }
        return alive;
    }

    public Queue<boolean[][]> generationsCreator(int universeSize, int genNum) {

        Queue <boolean[][]> gens = new LinkedList<boolean[][]>();
        Random random = new Random();
        boolean[][] currentGeneration = new boolean[universeSize][universeSize];
        for (int i = 0; i < universeSize; i++) {
            for (int j = 0; j < universeSize; j++) {
                currentGeneration[i][j] = random.nextBoolean();
            }
        }
        gens.add(currentGeneration);

        for(int h = 1; h <= genNum; h ++) {
            boolean[][] nextGeneration = new boolean[universeSize][universeSize];
            for (int i = 0; i < universeSize; i++) {
                for (int j = 0; j < universeSize; j++) {
                    int aliveNeighbors = getAliveNeighbors(currentGeneration, i, j);
                    if (currentGeneration[i][j]) {
                        nextGeneration[i][j] = aliveNeighbors == 2 || aliveNeighbors == 3;
                    } else {
                        nextGeneration[i][j] = aliveNeighbors == 3;
                    }
                }
            }
            gens.add(nextGeneration);
            currentGeneration = nextGeneration;
        }


        return gens;
    }

    private static int getAliveNeighbors(boolean[][] currentGeneration, int i, int j) {
        int aliveNeighbors = 0;
        int max = currentGeneration.length - 1;
        int yUp = i - 1 < 0 ? max : i - 1;
        int yDown = i + 1 > max ? 0 : i + 1;
        int xLeft = j - 1 < 0 ? max : j - 1;
        int xRight = j + 1 > max ? 0 : j + 1;

        if (currentGeneration[yUp][xLeft]) aliveNeighbors++;
        if (currentGeneration[yUp][j]) aliveNeighbors++;
        if (currentGeneration[yUp][xRight]) aliveNeighbors++;

        if (currentGeneration[i][xLeft]) aliveNeighbors++;
        if (currentGeneration[i][xRight]) aliveNeighbors++;

        if (currentGeneration[yDown][xLeft]) aliveNeighbors++;
        if (currentGeneration[yDown][j]) aliveNeighbors++;
        if (currentGeneration[yDown][xRight]) aliveNeighbors++;
        return aliveNeighbors;
    }
}

import java.awt.*;

import java.awt.event.ActionEvent;

import java.awt.event.ActionListener;

import java.util.Hashtable;

import javax.swing.*;

import javax.swing.event.ChangeEvent;

import javax.swing.event.ChangeListener;

import java.awt.Dimension;

public class ControlPanel extends JPanel implements ActionListener, ChangeListener {

    static GameOfLifeAlgo gl;
    public static JToggleButton playToggleButton;
    public static JButton resetButton;
    static JLabel generationLabel;
    static JLabel aliveLabel;
    static JLabel speedMode;
    public static JSlider evolutionSpeedSlider;
    Hashtable<Integer, JLabel> labelTable;

    static int minSpeed = 0;
    static int maxSpeed = 2000;
    static int initSpeed = 800;
    public static boolean paused = false;
    public static boolean reseted = false;
    static int evolutionSpeed = initSpeed;

    public  ControlPanel() {
        setLayout(new BoxLayout(this, BoxLayout.Y_AXIS));
        add(Box.createVerticalStrut(5));
        playToggleButton = new JToggleButton("Pause");
        playToggleButton.setName("PlayToggleButton");

        playToggleButton.addActionListener(this);

        add(playToggleButton);
        add(Box.createVerticalStrut(5));

        resetButton = new JButton("Reset");
        resetButton.setName("ResetButton");


        resetButton.addActionListener(e -> {
            reseted = true;

        });
        add(resetButton);
        add(Box.createVerticalStrut(5));

        generationLabel = new JLabel("Generation #0");
        generationLabel.setName("GenerationLabel");
        add(generationLabel);

        aliveLabel = new JLabel("Alive: 0");
        aliveLabel.setName("AliveLabel");
        add(aliveLabel);
        add(Box.createVerticalStrut(5));

        speedMode = new JLabel("Speed mode");
        speedMode.setName("speedMode");
        add(speedMode);
        add(Box.createVerticalStrut(5));

        evolutionSpeedSlider = new JSlider(JSlider.HORIZONTAL, minSpeed, maxSpeed, initSpeed);

        setPreferredSize(new Dimension(130, 20));
        evolutionSpeedSlider.addChangeListener( this);
        evolutionSpeedSlider.setMajorTickSpacing(400);
        evolutionSpeedSlider.setPaintTicks(true);
        labelTable = new Hashtable<Integer, JLabel>();
        labelTable.put(minSpeed, new JLabel("Fast"));
        labelTable.put(maxSpeed, new JLabel("Slow"));
        evolutionSpeedSlider.setLabelTable(labelTable);
        evolutionSpeedSlider.setPaintLabels(true);
        add(evolutionSpeedSlider);



    }


    @Override
    public void actionPerformed(ActionEvent e) {
        JToggleButton btn = (JToggleButton) e.getSource();

        if (playToggleButton == btn) {
            if(playToggleButton.isSelected()) {
                playToggleButton.setText("Resume");
                paused = true;
            } else {
                playToggleButton.setText("Pause");
                paused = false;
            }
        }
    }


  //  @Override
    public void stateChanged(ChangeEvent e) {
        JSlider source = (JSlider)e.getSource();

        evolutionSpeed  =  evolutionSpeedSlider.getValue();
    }
}

import java.awt.Color;

import java.awt.GridLayout;

import javax.swing.JLabel;

import javax.swing.JPanel;

public class GenerationPanel extends JPanel {
    static JLabel[][] labels;


    public GenerationPanel(int universeSize, int genNum) {

        setLayout(new GridLayout(universeSize, universeSize, 0, 0));

        labels = new JLabel[universeSize][universeSize];

        for (int i = 0; i < universeSize; i++) {
            for (int j = 0; j < universeSize; j++) {
                labels[i][j] = new JLabel();
                labels[i][j].setBackground(Color.BLACK);
                add(labels[i][j]);
            }
        }
    }
}
