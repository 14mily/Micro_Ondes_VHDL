# Projet Conception Circuit Numérique (CoCiNum) - Système de Contrôle d'un Micro-Onde

## Auteurs
**Émilie GNASSINGBE - Kenneth NONVIGNON**  
**E3e - B3 (2024 - 2025)**

## Encadrement
**Référent Pédagogique & Encadrant : Valentin ARBOUX**

---

## Table des Matières
1. [Introduction](#introduction)
2. [Cahier des Charges](#cahier-des-charges)
3. [Programmation en VHDL](#programmation-en-vhdl)
4. [Tests & Résultats](#tests--résultats)
5. [Problèmes Rencontrés et Solutions Apportées](#problèmes-rencontrés-et-solutions-apportées)
6. [Conclusion](#conclusion)

---

## Introduction

### Contexte
Ce projet s'inscrit dans le cadre des travaux pratiques de l'ECUE **Conception Circuit Numérique (CoCiNum)**. L'objectif était de mettre en application les connaissances acquises lors du semestre 5 à travers la conception, la simulation et la réalisation de circuits numériques. Nous avons eu à concevoir, simuler et réaliser le **système de contrôle d'un micro-onde** en **VHDL** sur une carte **BASYS3**.

### Objectif du Projet
Nous avons dû concevoir un **système de contrôle de micro-onde en VHDL** sur une carte **BASYS3**. Ce système devait simuler les fonctions principales d'un micro-onde, notamment :
- La mise en marche
- La cuisson
- La fin de cuisson

---

## Cahier des Charges

Nous avons suivi un cahier des charges précis pour implémenter les fonctionnalités suivantes :

- **Simulation du buzzer** : Utilisation d'une LED pour marquer la fin du temps de cuisson.
- **Simulation du magnétron** : Utilisation d'une LED clignotante pour simuler l'émission des ondes.
- **Simulation de la porte fermée** : La cuisson ne peut commencer que si la porte est fermée et s'arrête si elle s'ouvre.
- **Sélection du temps de fonctionnement** : Configuration du temps de cuisson par dizaines de minutes, minutes, dizaines de secondes et secondes via des interrupteurs.
- **Simulation du bouton central** : Activation de la cuisson par un bouton "start".
- **Simulation du décompte (Afficheur 7 segments)** : Affichage du temps restant sur un afficheur 7 segments.


## Programmation en VHDL

Nous avons modifié un **StopWatch** fourni en travaux pratiques afin de :
- Transformer un chronomètre en **minuteur**.
- Implémenter une **machine à états** pour gérer les transitions (ATTENTE, M_ON, PAUSE, M_OFF).
- Implémenter un système de **pause et reprise** de la cuisson.
- Charger dynamiquement le temps de cuisson avant démarrage.

**Voici le code final du système :**

```cpp
library ieee;
use ieee.std_logic_1164.all;
use ieee.numeric_std.all;
    
entity Decrescendo is
  generic(
      N : positive 
  );

  port(
      clk_i, reset_i, dec_i            : in  std_logic;
      valeur_d_entree                  : in  integer range 0 to N - 1 := 0;
      la_sortie                        : out integer range 0 to N - 1;
      cycle_o                          : out std_logic
  );
end Decrescendo;
    
architecture Behavioral of Decrescendo is
  signal value_next   : integer range 0 to N - 1;
  signal value_reg    : integer range 0 to N - 1 := 0;
  signal load_i       : std_logic := '0';
begin
    
    
  -- PROCESSUS PRINCIPAL DE DÉCOMPTE
  K_M_p_value_reg : process(clk_i, reset_i)
begin
  if reset_i = '1' then
  -- Valeur minimale par défaut au reset
      value_reg <= 0;
  elsif rising_edge(clk_i) then
      if valeur_d_entree /= 0 then
          value_reg <= valeur_d_entree; -- Charger la valeur depuis valeur_d_entree
      elsif dec_i = '1' then
          value_reg <= value_next; -- Décompte normal
      end if;
  end if;
end process K_M_p_value_reg;
    
    
  -- LOGIQUE DE DÉCOMPTE
  value_next <= N - 1 when value_reg = 0 else value_reg - 1;
    
  -- ASSIGNATION DES SORTIES
  la_sortie <= value_reg;
  cycle_o <= '1' when dec_i = '1' and value_reg = 0 else '0';

end Behavioral;





Code du Micro-onde
-- Notre but est simple : réaliser de façon ultra simplifiée, le comportement d'un micro-onde.       
        library ieee; -- Elles vont nous permettre de manipuler les types standards en logique numérique ainsi que les portes logiques 
        use ieee.std_logic_1164.all; 
        use ieee.numeric_std.all; -- Nous a servi à un moment pour faire des conversions 
        use ieee.std_logic_textio; -- Nous permet d'écrire des valeurs logiques dans des fichiers ou la console.
       
       
 -- L'entité principale a pour nom SupremeOven 
  entity SupremeOven is
      port(
-- Elle aura besoin d      e la clk, des boutons ainsi que de quelques ports spécifiques aux entités que nous utiliserons dans l'architecture
       clk_i                                           : in  std_logic;
       btn_center_i, btn_right_i, btn_up_i, btn_down_i : in  std_logic := '0';


       switches_i             : in  std_logic_vector(15 downto 0):= "0000000000000000";
       leds_o                 : out std_logic_vector(15 downto 0):= "0000000000000000";
       disp_segments_n_o      : out std_logic_vector(0 to 6);
       disp_point_n_o         : out std_logic;
       disp_select_n_o        : out std_logic_vector(3 downto 0) := "0000"
            );
  end SupremeOven;
        
  architecture Structural of SupremeOven is
  -- Nous définissons les différents signaux (des fils électriques en somme) qui vont nous permetttre de relier les blocs fonctionnels entre eux 
    signal cycle_5ms, cycle_20ms, cycle_100ms,cycle_1sec, cycle_10sec, cycle_1min, cycle_10min, dec_100ms : std_logic;
          
    -- Le signal "marche" en particulier va nous servir à provoquer volontairement un arrêt dans le processus de décompte. Vous verrez bien;
    signal marche                                           : std_logic := '0';
          
    -- Nous avons 4 afficheurs. Afin de les parcourir aisément, nous nous sommes intelligemment servi de la sortie du diviseur 20ms.
    signal digit_index                                      : integer range 0 to 3;


    signal digit_100ms, digit_1sec, digit_1min, digit       : integer range 0 to 9;
    signal digit_10sec, digit_10min                         : integer range 0 to 5;
          
    -- Ce signal spécifique à l'entité SegmentDecoder nous permet d'assigner à chaque chiffre, les leds correspondantes devant être allumées
    signal segments                                         : std_logic_vector(0 to 6);
          
          
    -- La définition des états de notre machine à états est une étape cruciale dans l'écriture du code
    -- Nous avons donc les 4 états suivants. 
    type state_t is (ATTENTE, M_ON, PAUSE, M_OFF);
    signal current_state, next_state                        : state_t;
    -- Les deux signaux ci-dessus peuvent prendre les différents états de notre machine 
           
    -----------------------------------------------------------------------------------
           
    -- Les signaux suivants vont nous servir à récupérer les valeurs voulues pour le temps de cuisson
    -- Une fois fait, ces valeurs seront injectés dans les registres respectifs des compteurs 
    signal load_v_1, load_v_3 : integer range 0 to 9 := 0; -- Unités des secondes et des minutes
    signal load_v_2, load_v_4 : integer range 0 to 5 := 0; -- Dizaines des secondes et des minutes
            
            
    -- Ces deux signaux servent à détecter les fronts montants
    -- L'utilisation de l'EventDetector s'étant soldé par des échecs, nous avons pensé à refaire le concept à notre façon
    signal btn_up_prev, btn_down_prev : std_logic := '0'; 
    -----------------------------------------------------------------------------------
     alias PORTE : std_logic is switches_i(5);
     alias START : std_logic is btn_center_i;

     alias MAGNETRON : std_logic is leds_o(13);
     alias BUZZER : std_logic is leds_o(15);
            
  begin
        
  -- Nous avons 8 instances de notre entité Decrescendo dont 3 diviseurs de clock et 5 compteurs
        
   K_M_divider_5ms_inst : entity work.Decrescendo
          generic map(

              N => 5--00e3
              -- Cela nous permet d'avoir le cycle_5ms qui passe de 0 à 1 toutes les 5ms 
          )
          port map(
              clk_i   => clk_i,
              reset_i => '0',
              dec_i   => '1',
              valeur_d_entree => 0,
              la_sortie => open,
              cycle_o => cycle_5ms
          );
        
      K_M_divider_20ms_inst : entity work.Decrescendo
          generic map(
              N => 4
              -- Cela nous permet d'avoir le cycle_20ms qui passe de 0 à 1 toutes les 20ms 
          )
          port map(
              clk_i   => clk_i,
              reset_i => '0',
              dec_i   => cycle_5ms,
              valeur_d_entree => 0,
              la_sortie => digit_index,
              cycle_o => cycle_20ms
          );
        
      K_M_divider_100ms_inst : entity work.Decrescendo
          generic map(
              N => 5
              -- Cela nous permet d'avoir le cycle_100ms qui passe de 0 à 1 toutes les 100ms 
          )
          port map(
              clk_i   => clk_i,
              reset_i => '0',
              dec_i   => cycle_20ms,
              valeur_d_entree => 0,
              la_sortie => open,
              cycle_o => cycle_100ms
          );
        
      K_M_decounter_10x100ms_inst : entity work.Decrescendo
          generic map(
              N => 10
              -- On compte jusqu'à 10 afin d'avoir 1s
          )
          port map(
              clk_i   => clk_i,
              reset_i => btn_right_i,

              dec_i   => dec_100ms,
              valeur_d_entree => 0,
              la_sortie => digit_100ms,
              cycle_o => cycle_1sec

          );
        
      K_M_decounter_10x1sec_inst : entity work.Decrescendo
          generic map(
              N => 10
              -- On compte jusqu'à 10 afin d'avoir 10s
          )
          port map(
              clk_i   => clk_i,
              reset_i => btn_right_i,
              dec_i   => cycle_1sec,
              valeur_d_entree => load_v_1,
              la_sortie => digit_1sec,
              cycle_o => cycle_10sec
          );
        
      K_M_decounter_6x10sec_inst : entity work.Decrescendo
          generic map(
              N => 6
              -- On compte jusqu'à 6 afin d'avoir 60s(1 minute)
          )
          port map(
              clk_i   => clk_i,
              reset_i => btn_right_i,
              dec_i   => cycle_10sec,
              valeur_d_entree => load_v_2,
              la_sortie => digit_10sec,
              cycle_o => cycle_1min
          );
        
      K_M_decounter_10x1min_inst : entity work.Decrescendo
          generic map(
              N => 10
              -- On compte jusqu'à 10 afin d'avoir 10 minutes
          )
          port map(
              clk_i   => clk_i,
              reset_i => btn_right_i,
              dec_i   => cycle_1min,
              valeur_d_entree => load_v_3,
              la_sortie => digit_1min,
              cycle_o => cycle_10min
          );
                
          K_M_decounter_6x10min_inst : entity work.Decrescendo

          generic map(
              N => 6
              -- On compte jusqu'à 6 afin d'avoir 60 minutes
          )
          port map(
              clk_i   => clk_i,
              reset_i => btn_right_i,
              dec_i   => cycle_10min,
              valeur_d_entree => load_v_4,
              la_sortie => digit_10min,
              cycle_o => open
          );
  -- Notre microonde est donc en mesure de faire un décompte d'une heure.
  -- A tous les coups, votre plat sera forcément chaud.
        
      K_M_decoder_inst : entity work.SegmentDecoder
          port map(

        -- digit_i permet d'indiquer qu'est ce qui sera affiché selon l'afficheur sélectionné
              digit_i    => digit,
              -- Ici, les ports des segments sont connectés aux fils nommés segments
              segments_o => segments
          );
        
      -- Cette ligne est certainement la plus importante. C'est elle qui va permettre de mettre le décompteur en marche ou non. D'où son nom. 
      dec_100ms <= cycle_100ms and marche;
            
  process(clk_i)
  begin
      if rising_edge(clk_i) then
      -- Puisque digit_index varie entre 0 et 3, il tombe à point nommé
          case digit_index is
              when 0 => disp_select_n_o <= "1110"; -- Sert à activer le premier afficheur 
              when 1 => disp_select_n_o <= "1101"; -- Sert à activer le deuxième afficheur
              when 2 => disp_select_n_o <= "1011"; -- Sert à activer le troisième afficheur
              when 3 => disp_select_n_o <= "0111"; -- Sert à activer le quatrième afficheur
              when others => disp_select_n_o <= "1111"; -- On désactive tous les afficheurs
          end case;
      end if;
  end process;
        
  -- Voilà enfin notre machine à états
      process(clk_i, btn_right_i, current_state)

      begin
      
      -- On va commencer par définir la valeur initiale d'un peu tout ce qu'on va utiliser 
      leds_o(12)  <= '0'; -- La led de l'état ATTENTE
      MAGNETRON  <= '0'; -- La led de l'état M_ON
      leds_o(14) <= '0';  -- La led de l'état PAUSE
      BUZZER <= '0';  -- La led de l'état M_OFF
            
      leds_o(0) <= '0'; -- La led de la seconde 0
      leds_o(1) <= '0'; -- La led de la seconde 1
      leds_o(2) <= '0'; -- La led de la seconde 2
      leds_o(3) <= '0'; -- La led de la seconde 3
      leds_o(4) <= '0'; -- La led de la seconde 4
      leds_o(5) <= '0'; -- La led de la seconde 5
      leds_o(6) <= '0'; -- La led de la seconde 6
      leds_o(7) <= '0'; -- La led de la seconde 7
      leds_o(8) <= '0'; -- La led de la seconde 8
      leds_o(9) <= '0'; -- La led de la seconde 9
            
            
          if btn_right_i = '1' then
          -- Si on reset, alors on retourne en état attente
              current_state <= ATTENTE;
              marche <= '0';  -- Initialisation
          elsif rising_edge(clk_i) then
          -- Sinon, si le bouton du reset n'est pas actif au prochain front montant, on passe à l'état suivant
              if btn_right_i = '0' then
              current_state <= next_state;
                    
              case current_state is
        
              -- Si nous sommes en ATTENTE, 
                  when ATTENTE =>
                      leds_o(12) <= '1'; -- On allume la led qui lui correspond

                      -- On s'occupe de la gestion du temps de cuisson uniquement dans l'état ATTENTE
         ---------------------------------------------------------------------------------------- 
          -- On gère de façon indépendante chacun d'entre eux (unités des secondes)
          if switches_i(0) = '1' then
              if btn_up_i = '1' and btn_up_prev = '0' and load_v_1 < 9 then
                  load_v_1 <= load_v_1 + 1;
              elsif btn_down_i = '1' and btn_down_prev = '0' and load_v_1 > 0 then
                  load_v_1 <= load_v_1 - 1;
              end if;
          end if;
          -- on s'assure d'incrémenter ou de décrémenter le temps sans pour autant déborder
                
          -- (dizaines des secondes)
          if switches_i(1) = '1' then
              if btn_up_i = '1' and btn_up_prev = '0' and load_v_2 < 5 then
                  load_v_2 <= load_v_2 + 1;
              elsif btn_down_i = '1' and btn_down_prev = '0' and load_v_2 > 0 then
                  load_v_2 <= load_v_2 - 1;
              end if;
          end if;
        
          -- (unités des minutes)
          if switches_i(2) = '1' then
              if btn_up_i = '1' and btn_up_prev = '0' and load_v_3 < 9 then
                  load_v_3 <= load_v_3 + 1;
              elsif btn_down_i = '1' and btn_down_prev = '0' and load_v_3 > 0 then
                  load_v_3 <= load_v_3 - 1;
              end if;
          end if;
                
          -- (dizaines des minutes)
          if switches_i(3) = '1' then
              if btn_up_i = '1' and btn_up_prev = '0' and load_v_4 < 5 then
                  load_v_4 <= load_v_4 + 1;
              elsif btn_down_i = '1' and btn_down_prev = '0' and load_v_4 > 0 then
                  load_v_4 <= load_v_4 - 1;
              end if;
          end if;
                
          -- On met à jour afin de s'assurer que cela ne marche que lors d'un front montant
          btn_up_prev   <= btn_up_i;
          btn_down_prev <= btn_down_i;
         -------------------------------------------------------------
          if PORTE = '1' and START = '1' then
          -- Si la porte est fermée et que le bouton start est enfoncé, on sort de l'attente pour entrer dans l'état M_ON
             next_state <= M_ON;
          else
          -- Sinon, on reste dans l'état ATTENTE
             next_state <= ATTENTE; -- Rester dans ATTENTE
          end if;
   
                  -- A présent, on entre dans l'état M_ON 
                  when M_ON =>
              -- On réinitialise les valeurs 
              load_v_1 <= 0;
              load_v_2 <= 0;
              load_v_3 <= 0;
              load_v_4 <= 0;


        
              -- La structure conditionnelle suivante va servir à ajouter un petit effet visuel
              -- Chaque led s'allume en même temps que l'unité des secondes correspondantes 
              if digit_1sec = 0 then 
               leds_o(0) <= '1'; 
              -------------------------
              elsif digit_1sec = 1 then
                  leds_o(1) <= '1';
              elsif digit_1sec = 2 then
                  leds_o(2) <= '1';
              elsif digit_1sec = 3 then
                  leds_o(3) <= '1';
              elsif digit_1sec = 4 then
                  leds_o(4) <= '1';
              elsif digit_1sec = 5 then
                  leds_o(5) <= '1';
              elsif digit_1sec = 6 then
                  leds_o(6) <= '1';
              elsif digit_1sec = 7 then
                  leds_o(7) <= '1';
              elsif digit_1sec = 8 then
                  leds_o(8) <= '1';
              elsif digit_1sec = 9 then
                  leds_o(9) <= '1';
              -------------------------    
              end if;
              ---------------------------------------------
                  MAGNETRON <= '1'; -- La led correspondant à l'état M_ON 
                  marche <= '1';  -- Mise en marche du compteur
                  if digit_1sec = 0 and digit_10sec = 0 and digit_1min = 0 and digit_10min = 0 then
                      next_state <= M_OFF;  -- Passage à l'état OFF si les compteurs sont à 0
                  elsif PORTE = '0' then
                      next_state <= PAUSE;  -- Mettre en pause si l'interrupteur est relâché
                  else
                      next_state <= M_ON;  -- Rester dans M_ON
                  end if;
        
              -- État PAUSE : compteur inactif, attendre un redémarrage
              when PAUSE =>
                  leds_o(14) <= '1';
                  marche <= '0';  -- Arrêter le compteur
                  if PORTE = '1' then
                      next_state <= M_ON;  -- Reprendre le comptage
                  else
                      next_state <= PAUSE; -- Rester dans PAUSE
                  end if;
        
              -- État M_OFF : état final (arrêt complet)
              when M_OFF =>
                  BUZZER <= '1';
                  marche <= '0';  -- État inactif
                  next_state <= M_OFF;  -- Boucle sur lui-même pour rester stable
     
              -- Cas de sécurité (others)
                  when others =>
                      leds_o(12)  <= '0'; -- la led de l'état stop
                      MAGNETRON  <= '0';
                      leds_o(14) <= '0';
                      BUZZER <= '0';
                            
                      leds_o(0) <= '0';

                      leds_o(1) <= '0';
                      leds_o(2) <= '0';
                      leds_o(3) <= '0';
                      leds_o(4) <= '0';
                      leds_o(5) <= '0';
                      leds_o(6) <= '0';
                      leds_o(7) <= '0';
                      leds_o(8) <= '0';
                      leds_o(9) <= '0';
                      marche <= '0';
                      next_state <= ATTENTE; -- Retour à l'état initial pour éviter des erreurs
                        
         
        
          end case;
          current_state <= next_state; -- Mise à jour de l'état courant
              end if;
          --else 
              --current_state <= ATTENTE;
          end if; 
      end process;
        --------------------------------------------------------------    
      disp_segments_n_o <= not segments;
      disp_point_n_o    <= '0';
            
        ------------------------------------------------------------------------------
  process(current_state, digit_index)
  begin
      if current_state = ATTENTE then 
                       case digit_index is
                      when 0 => digit <= load_v_1; -- Afficheur 1
                      when 1 => digit <= load_v_2; -- Afficheur 2
                      when 2 => digit <= load_v_3; -- Afficheur 3
                      when 3 => digit <= load_v_4; -- Afficheur 4

                      when others => digit <= 0;  -- Sécurité
                        end case;
        
      else 
                      case digit_index is
              when 0 => digit <= digit_1sec; -- Afficheur 1
              when 1 => digit <= digit_10sec; -- Afficheur 2
              when 2 => digit <= digit_1min; -- Afficheur 3
              when 3 => digit <= digit_10min; -- Afficheur 3
              when others => digit <= 0;  -- Sécurité
          end case;
       end if;
  end process;
        
  end Structural;
```

---

## Tests & Résultats

### États Clés

1. **Passage ATTENTE → M_ON** :
   
   ![image](https://github.com/user-attachments/assets/09becea0-c6b4-4a0f-84bd-5e59fd47fae7)

   - La valeur **load_v_3 = 1** est chargée dans le registre du minuteur.
   - Si la porte est fermée et le bouton "start" enfoncé, la cuisson commence (**marche = 1**).

2. **Passage M_ON → PAUSE** :

   ![image](https://github.com/user-attachments/assets/aa86a0ca-730d-4bde-8e0f-6455302fabe7)

   - L'ouverture de la porte interrompt la cuisson (**marche = 0**).

3. **Passage PAUSE → M_ON** :

   ![image](https://github.com/user-attachments/assets/a0a5c9b9-9101-428f-92f7-6e6e850e14c5)

   - La fermeture de la porte reprend la cuisson (**marche = 1**).

4. **Passage M_ON → M_OFF** :

    ![image](https://github.com/user-attachments/assets/2ffc6a72-8546-429a-aa91-8a702fb96017)

   - Lorsque tous les compteurs atteignent **0**, la cuisson s'arrête (**marche = 0**).

---

## Problèmes Rencontrés et Solutions Apportées

1. **Transformation d'un chronomètre en minuteur** :
   - Modification des lignes de code du **CounterModN** pour **décrémenter** au lieu d'incrémenter.
   
```cpp
value_next <= N - 1 when value_reg = 0 else value_reg - 1;
cycle_o <= '1' when dec_i = '1' and value_reg = 0 else '0';
```

2. **Mise en pause et reprise du décompte** :
   - Utilisation d'un **signal std_logic** contrôlé manuellement au lieu de l'interrupteur **switches_i(15)**.

3. **Arrêt du minuteur à 0** :
   - Implémentation d'une **machine à états** pour gérer le signal "marche".

4. **Fixation du temps de cuisson** :
   - Utilisation d'un port **load_v** permettant de charger dynamiquement le temps choisi avant le lancement de la cuisson dans l'état **ATTENTE**.

---

## Conclusion

Ce projet nous a permis de mettre en pratique nos connaissances en **VHDL** et d'acquérir de l'expérience en conception et simulation de circuits numériques.

Nous avons consolidé nos compétences en :

- **Programmation VHDL** sur carte **BASYS3.**
- **Analyse et modification d'une architecture existante.**
- **Utilisation d'une machine à états** pour la gestion des processus.
  
Grâce aux bancs de tests, nous avons validé la robustesse du système et surmonté divers défis techniques.

---

## 📌 Technologies utilisées

- **VHDL**
- **Carte FPGA BASYS3**
- **Vivado (Xilinx)**

---

1. Charger le projet sur **Vivado**.
2. Compiler et synthétiser le code.
3. Générer le bitstream et programmer la **BASYS3**.
