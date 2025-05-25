import React, { createContext, useContext, useEffect, useState } from "react";
import axios from 'axios'
import { CoinList } from "./config/api";
import { onAuthStateChanged } from "firebase/auth";
import { auth } from "./firebase";
import { doc } from "firebase/firestore/lite";
import { onSnapshot } from "firebase/firestore"
import { db } from "./firebase";

const Crypto = createContext();

const CryptoContext = ({ children }) => {
  const [currency, setCurrency] = useState("INR");
  const [symbol, setSymbol] = useState("₹");
  const [coins, setCoins] = useState([]);
  const [loading, setLoading] = useState(false);
  const [user, setUser] = useState(null); 
  const [alert, setAlert] = useState({
    open:false,
    message:"",
    type:"success",
  
  })
  const [watchlist, setWatchlist] = useState([]);

  useEffect(() => {
    if (user) {
      const coinRef = doc(db, "watchlist", user.uid);
      const unsubscribe = onSnapshot(coinRef, (coin) => {
        if (coin.exists()) {
          setWatchlist(coin.data().coins);
        } else {
          console.log("No items in watchlist");
          setWatchlist([]); // Clear the watchlist if no items exist
        }
      }, (error) => {
        console.error("Error fetching watchlist: ", error);
      });
  
      return () => {
        unsubscribe(); // Clean up the listener on unmount
      };
    } else {
      setWatchlist([]); // Clear the watchlist if no user is logged in
    }
  }, [user]);
  
  useEffect(() => {
   onAuthStateChanged(auth,(user) => {n
     if(user)setUser(user);
     else setUser(null);

     console.log(user);
   });

  }, [ ]);
  
  const fetchCoins = async () => {
    setLoading(true);
    const { data } = await axios.get(CoinList(currency));
    console.log(data);

    setCoins(data);
    setLoading(false);
  };

  
  useEffect(() => {
    if (currency === "INR") setSymbol("₹");
    else if (currency === "USD") setSymbol("$");
  }, [currency]);

  return (
    <Crypto.Provider value={{ currency, setCurrency, symbol, coins, loading,fetchCoins,alert ,setAlert,user,watchlist }}>
      {children}
    </Crypto.Provider>
  );
};

export default CryptoContext;

export const CryptoState = () => {
  return useContext(Crypto);
};
