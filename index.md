## This is the website for RASSLE attack

For more details on the paper, you can follow this link -> (https://tches.iacr.org/index.php/TCHES/article/view/8795/8395) 


### Short Summary of the attack

In this attack, we show that the OpenSSL implementation of scalar multiplication on P-256 curve has varying number of add-and-sub function calls, which depends on the secret scalar bits. We demonstrate through several experiments that the number of add-and-sub function calls can be used to template the secret bit, which can be picked up by the spy using the principles of RASSLE. We eventually demonstrate a full end-to-end attack on OpenSSL ECDSA using curve parameters of curve P-256.  We particularly abuse the deadline scheduler policy to attain perfect synchronization between the spy and victim, without any aid of induced synchronization from the victim process. This synchronization and timing leakage through RASSLE  is sufficient to retrieve theMost Significant Bits (MSB) of the ephemeral nonces used while signature generation, from which we subsequently retrieve the secret signing key of the sender applying the Hidden Number Problem.





# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/anirbanc3/rassle.github.io/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
