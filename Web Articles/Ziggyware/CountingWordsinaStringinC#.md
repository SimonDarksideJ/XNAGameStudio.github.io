# Counting Words in a String in C#


In this example we use a sorted list to count the number of distinct words and total character count of a string in C# :)

```csharp
#region WordCounter
    public class WordCounter
    {
        string testString = null;
        SortedList wordCounter = new SortedList();

        public WordCounter(string str)
        {
            testString = str;
        }

        public int UniqueWords
        {
            get 
            {
                return wordCounter.Count;
            }
        }

        public IDictionaryEnumerator GetWordsAlphabeticallyEnumerator()
        {
            return wordCounter.GetEnumerator();
        }

        public IDictionaryEnumerator GetWordsByOccurrenceEnumerator()
        {
            SortedList sl = new SortedList();
            IDictionaryEnumerator enumer = GetWordsAlphabeticallyEnumerator();
            while(enumer.MoveNext())
            {
                WordOccurrence wo = new WordOccurrence((int)enumer.Value,(string)enumer.Key);
                sl.Add(wo,null);
            }
            return sl.GetEnumerator();
        }

        public bool CountStats(out int numWords,out int numChars)
        {
            numWords = 0;
            numChars = 0;
            string[] words = testString.Split(null);



            foreach(string str in words)
            {
                if(str.Length > 0)
                {
                    numChars += str.Length;
                    if(!wordCounter.ContainsKey(str))
                    {
                        wordCounter.Add(str,1);
                        numWords++;
                    }
                    else
                    {
                        int iWordCount = (int)wordCounter[str];
                        wordCounter[str] = iWordCount+1;
                    }

                }
            }

            return true;
        }

        public class WordOccurrence : IComparable
        {
            int occurences;
            string word;

            public WordOccurrence(int occurrences,string word)
            {
                this.occurences = occurences;
                this.word = word;
            }



            public int Occurrences
            {
                get {return occurences;}
            }

            public string Word
            {
                get { return word; }
            }
    #region IComparable Members

            public int CompareTo(object o)
            {
                return string.Compare(word,((WordOccurrence)o).word);
            }

    #endregion
        }
    }
#endregion
```

Here is an implementation of the class: 

```csharp
    string strWords = @"
    Hello world
    hello here      i am where are you hello you";

    WordCounter wc = new WordCounter(strWords);

    int numWords,numChars;
    wc.CountStats(out numWords,out numChars);

    Console.WriteLine("Num Words: {0} Num Chars: {1}",numWords,numChars);

    IDictionaryEnumerator enumer = wc.GetWordsAlphabeticallyEnumerator();
    while(enumer.MoveNext())
    {
        Console.WriteLine(enumer.Key);
    }

    enumer = wc.GetWordsByOccurrenceEnumerator();
    while(enumer.MoveNext())
    {
        Console.WriteLine("Word: {0}",((WordCounter.WordOccurrence)enumer.Key).Word);
    }
```