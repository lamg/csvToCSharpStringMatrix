# Convert a CSV table to a C# source code string matrix

### Abstract

This project comes out from the necessity of testing a parser written in C# without relying on any IO operation. This practice is a standard in testing, since it allows the isolation of parsing errors from IO errors or any other error in a complex library to get the data.

### The program

The converter reads a CSV (separated by semicolon) from standard input and writes the C# source code string matrix to standard output. Errors are written to standard error. In the conversion, every element of the CSV table must be quoted (surrounded with ") if is not already. Every row of the CSV table represents a row in the C# source code string
matrix, thus it must be sorrounded with { and }. Finally, the set of rows is surrounded with { and }.

``` haskell
main :: IO ()
main = getInput >>= return . convert >>= either putStderr putStr

putStderr :: String -> IO ()
putStderr = hPutStrLn stderr

getInput :: IO [String]
getInput = hGetContents stdin >>= return . lines
```


The `convert` function gets the input string splitted by newline characters.

``` haskell
convert :: [String] -> Either String String
convert = (unlines <$>) . sequence . surround (Right "{") (Right "}") . map toArray
```

The `surround` function adds its first parameter to the beginning of a list and its second parameter to the end of that list.

``` haskell
surround :: a -> a -> [a] -> [a]
surround x y l = x:l ++ [y]
```

The `toArray` function converts a String in CSV format (a row) to a C# source code string array.

``` haskell
toArray :: String -> Either String String
toArray s = return (splitBySemi s) >>= 
  sequence . map quote >>= 
  return . (++ ",") .surround '{' '}' . intercalate ","

splitBySemi :: String -> [String]
splitBySemi = map unpack . splitOn (pack ";") . pack
```

The `quote` function surrounds with double quotes a String if is not surrounded by double quotes.

 ``` haskell
quote :: String -> Either String String
quote [] = Right "\"\""
quote [x]|x == '"' = Left "Quotes not matching"
         |otherwise = Right $ surround '"' '"' [x]
quote (x:xs) = 
  if x == q || l == q then 
    if x == q && l == q then Right (x:xs)
    else Left "Quotes not matching"
  else Right $ surround q q (x:xs)
  where 
    l = last xs
    q = '"'
 ```


The above code works in a single file `Main.hs` with the following heading:

``` haskell
module Main where
import System.IO (hPutStrLn, stderr, stdin, hGetContents)
import Data.List (intercalate)
import Data.Text (pack, unpack, splitOn)
```
