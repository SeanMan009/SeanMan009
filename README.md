- 👋 Hi, I’m @SeanMan009
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...

<!---
SeanMan009/SeanMan009 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
import React, { useState, useEffect } from 'react';
import { SafeAreaView, StyleSheet, Text, FlatList, PermissionsAndroid, Platform, Button, View } from 'react-native';
import InsultInput from './src/components/InsultInput';
import AICompetitor from './src/components/AICompetitor';

const MAX_ROUNDS = 5;

const App = () => {
  const [gameState, setGameState] = useState('notStarted'); // 'notStarted', 'playerTurn', 'aiTurn', 'gameOver'
  const [insults, setInsults] = useState([]);
  const [currentRound, setCurrentRound] = useState(0);
  const [scores, setScores] = useState({ player: 0, ai: 0 });

  useEffect(() => {
    requestMicrophonePermission();
  }, []);

  const requestMicrophonePermission = async () => {
    if (Platform.OS === 'android') {
      try {
        const granted = await PermissionsAndroid.request(
          PermissionsAndroid.PERMISSIONS.RECORD_AUDIO,
          {
            title: 'Microphone Permission',
            message: 'This app needs access to your microphone for voice input.',
            buttonNeutral: 'Ask Me Later',
            buttonNegative: 'Cancel',
            buttonPositive: 'OK',
          },
        );
        if (granted !== PermissionsAndroid.RESULTS.GRANTED) {
          console.log('Microphone permission denied');
        }
      } catch (err) {
        console.warn(err);
      }
    }
  };

  const startGame = () => {
    setGameState('playerTurn');
    setCurrentRound(1);
    setInsults([]);
    setScores({ player: 0, ai: 0 });
  };

  const handleInsultSubmit = (insult) => {
    const newInsults = [...insults, { id: Date.now().toString(), text: insult, isPlayer: true }];
    setInsults(newInsults);
    setGameState('aiTurn');
    setTimeout(aiTurn, 1500);
  };

  const aiTurn = () => {
    const aiInsult = generateAIInsult();
    const newInsults = [...insults, { id: Date.now().toString(), text: aiInsult, isPlayer: false }];
    setInsults(newInsults);
    setCurrentRound(currentRound + 1);
    
    if (currentRound >= MAX_ROUNDS) {
      setGameState('gameOver');
    } else {
      setGameState('playerTurn');
    }

    // Simple scoring: AI always scores 1 point
    setScores(prevScores => ({ ...prevScores, ai: prevScores.ai + 1 }));
  };

  const generateAIInsult = () => {
    const insults = [
      "Your comebacks are as weak as your Wi-Fi signal.",
      "I've seen better insults in a kindergarten playground.",
      "Did you get your insult from a fortune cookie?",
      "Your wit is so dull, it couldn't cut through warm butter.",
      "I'm not saying you're boring, but you make vanilla look exciting.",
    ];
    return insults[Math.floor(Math.random() * insults.length)];
  };

  const renderInsult = ({ item }) => {
    if (item.isPlayer) {
      return <Text style={styles.playerInsult}>You: {item.text}</Text>;
    } else {
      return <AICompetitor insult={item.text} />;
    }
  };

  return (
    <SafeAreaView style={styles.container}>
      <Text style={styles.title}>Insults: A Witty Comeback Game</Text>
      {gameState === 'notStarted' && (
        <Button title="Start Game" onPress={startGame} />
      )}
      {gameState !== 'notStarted' && (
        <View style={styles.gameInfo}>
          <Text>Round: {currentRound}/{MAX_ROUNDS}</Text>
          <Text>Score - You: {scores.player} | AI: {scores.ai}</Text>
        </View>
      )}
      <FlatList
        data={insults}
        keyExtractor={(item) => item.id}
        renderItem={renderInsult}
      />
      {gameState === 'playerTurn' && (
        <InsultInput onSubmit={handleInsultSubmit} />
      )}
      {gameState === 'aiTurn' && (
        <Text style={styles.waitingText}>AI is thinking...</Text>
      )}
      {gameState === 'gameOver' && (
        <View style={styles.gameOver}>
          <Text style={styles.gameOverText}>Game Over!</Text>
          <Text>Final Score - You: {scores.player} | AI: {scores.ai}</Text>
          <Button title="Play Again" onPress={startGame} />
        </View>
      )}
    </SafeAreaView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#F5FCFF',
  },
  title: {
    fontSize: 20,
    fontWeight: 'bold',
    textAlign: 'center',
    margin: 10,
  },
  gameInfo: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    padding: 10,
  },
  playerInsult: {
    fontSize: 16,
    margin: 10,
    color: 'blue',
  },
  waitingText: {
    textAlign: 'center',
    margin: 10,
    fontStyle: 'italic',
  },
  gameOver: {
    alignItems: 'center',
    margin: 20,
  },
  gameOverText: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 10,
  },
});

export default App;
