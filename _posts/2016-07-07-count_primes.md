---
layout:            post
title:             "LeetCode 204 Count Primes"
date:              2016-07-07 08:00:00 +0800
tags:              OJ
category:          OJ
---

## Count Primes
*Count the number of prime numbers less than a non-negative number, n.*  
The following is $$O(n^2)$$ Solution  

    class Solution {
    public:
       vector<int> primes;
       int countPrimes(int n) {
           if (n == 0 || n == 1 || n == 2)
               return 0;
           primes.push_back(2);
           for (int i = 3; i < n; i += 2) {
               vector<int>::iterator it = primes.begin();
               for (; it != primes.end(); ++it) {
                   if (*it > (i / 2)) {
                       primes.push_back(i);
                       break;
                   }
                   if (!(i % *it))
                       break;
               }
           }
           return primes.size();
       }
    };
    
The following is $$O(n)$$ Solution  

    class Solution {
    public:
        int countPrimes(int n) {
            if (n < 2)
                return 0;
            vector<int> primes(n, false);
            int upper = sqrt(n);
            int counter = 0;
            for (int i = 2; i < n; ++i) {
                if (primes[i])
                    continue;
                ++counter;
                if (i > upper)
                    continue;
                for (int j = i * i; j < n; j += i)
                    primes[j] = true;
            }
            return counter;
        }
    };