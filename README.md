# 3blockchain-papildoma

# 1 papildoma
1. Įdiekite libbitcoin-system biblioteką: Sekite instrukcijas jūsų operacinei sistemai.
Pagal instrukcijas https://github.com/libbitcoin/libbitcoin-system įsidiegiau libbitcoin-system biblioteką macos OS:
<img width="565" alt="Screenshot_2024-11-12_at_00 43 24" src="https://github.com/user-attachments/assets/911001e8-8965-4a79-bdb3-287e04775bb7">

<img width="698" alt="Screenshot 2024-11-13 at 23 31 57" src="https://github.com/user-attachments/assets/ab1b99bc-e80b-4ce0-aee1-8b5d9111075a">

<img width="696" alt="Screenshot 2024-11-13 at 23 32 18" src="https://github.com/user-attachments/assets/f27c9b61-25a5-43fa-aece-754572590990">


2. Išanalizuokite pateiktą create_merkle() funkciją: Ši funkcija, parašyta C++ kalba, realizuoja Merkle medžio algoritmą.
Čia reikėjo pakeisti <bitcoin.bitcoin.hpp> į <bitcoin/system.hpp>
```
//  main.cpp
//  kakakakskas
//
//  Created by Kamilė Zobėlaitė on 2024-11-12.
//

#include <bitcoin/system.hpp>
#include <iostream>
bc::hash_digest create_merkle(bc::hash_list& merkle)
 {
     // Stop if hash list is empty or contains one element
     if (merkle.empty())
         return bc::null_hash;
     else if (merkle.size() == 1)
         return merkle[0];
     // While there is more than 1 hash in the list, keep looping...
     while (merkle.size() > 1)
     {
         // If number of hashes is odd, duplicate last hash in the list.
         if (merkle.size() % 2 != 0)
             merkle.push_back(merkle.back());
         // List size is now even.
         assert(merkle.size() % 2 == 0);
         // New hash list.
         bc::hash_list new_merkle;
         // Loop through hashes 2 at a time.
         for (auto it = merkle.begin(); it != merkle.end(); it += 2)
         {
             // Join both current hashes together (concatenate).
             bc::data_chunk concat_data(bc::hash_size * 2);
             auto concat = bc::serializer<
                 decltype(concat_data.begin())>(concat_data.begin());
             concat.write_hash(*it);
             concat.write_hash(*(it + 1));
             // Hash both of the hashes.
             bc::hash_digest new_root = bc::bitcoin_hash(concat_data);
             // Add this to the new list.
             new_merkle.push_back(new_root);
}
         // This is the new list.
         merkle = new_merkle;
         // DEBUG output -------------------------------------
         std::cout << "Current merkle hash list:" << std::endl;
         for (const auto& hash: merkle)
             std::cout << "  " << bc::encode_base16(hash) << std::endl;
         std::cout << std::endl;
         // --------------------------------------------------
}
     // Finally we end up with a single item.
     return merkle[0];
 }
int main() {
    bc::hash_list tx_hashes{{
        bc::hash_literal("8c14f0db3df150123e6f3dbbf30f8b955a8249b62ac1d1ff16284aefa3d06d87"),
        bc::hash_literal("fff2525b8931402dd09222c50775608f75787bd2b87e56995a7bdd30f79702c4"),
        bc::hash_literal("6359f0868171b1d194cbee1af2f16ea598ae8fad666d9b012c8ed2b79a236ec4"),
        bc::hash_literal("e9a66845e05d5abc0ad04ec80f774a7e585c6e8db975962d069a522137b80c1d"),
    }};
    const bc::hash_digest merkle_root = create_merkle(tx_hashes);
    std::cout << "Merkle Root Hash: " << bc::encode_base16(merkle_root) << std::endl;
//     std::cout << "Merkle Root Hash-2: " << bc::encode_hash(merkle_root) << std::endl;
    return 0;
}

```
3. Sukompiliuokite ir ištestuokite kodą:
`$ clang++ -std=c++11 -o merkle merkle.cpp $(pkg-config --cflags --libs libbitcoin)
$ ./merkle`
Čia jau teko daug padirbėti, nes buvo labai daaaaug error'ų. Bandžiau leisti paprastai per Xcode, bet iš pradžių nepavyko, nors ir buvo norodyta Header Search Path į /Users/kamilezobelaite/myprefix/include/bitcoin ir Library Search Path į /Users/kamilezobelaite/myprefix/lib, taip pat ir per terminalą.

<img width="558" alt="Screenshot 2024-11-13 at 23 36 42" src="https://github.com/user-attachments/assets/018b159a-7094-47b7-b2b1-7e8933a37edd">

Kaip matote neranda libbitcoin-system, nors jis YRA

<img width="954" alt="Screenshot 2024-11-13 at 23 38 12" src="https://github.com/user-attachments/assets/5c7bbf06-80ca-4880-b95c-c5eb5ec80c95">

<img width="957" alt="Screenshot 2024-11-13 at 23 38 26" src="https://github.com/user-attachments/assets/15a6ecdc-95b3-4ab0-85a4-4d0b59d3d079">

Tai ką dariau?
Radau naudingą šaltinį, kad reikia export'inti https://github.com/libbitcoin/libbitcoin-system/issues/808, tai tą ir dariau:

<img width="579" alt="Screenshot 2024-11-13 at 23 40 37" src="https://github.com/user-attachments/assets/7446c257-1332-47b0-8a83-2285fc1a9483">

`export PKG_CONFIG_PATH=/Users/kamilezobelaite/myprefix/lib/pkgconfig:$PKG_CONFIG_PATH` 

Tada pasižiūrėjau kokie yra flag'ai (bus naudinga Xcode sutvarkyti, kad veiktų)

`pkg-config --cflags --libs libbitcoin-system`

<img width="627" alt="Screenshot 2024-11-13 at 23 41 43" src="https://github.com/user-attachments/assets/73525ebb-7a48-4b29-8283-b011e1d23183">

Tada subuildinam:

 `g++ -std=c++11 -o bitcoin_test main.cpp $(pkg-config --cflags --libs libbitcoin-system)`

 Generuoja daug įspėjimų, bet svarbiausia klaidų jau nebėra:
 
 <img width="620" alt="Screenshot 2024-11-13 at 23 46 14" src="https://github.com/user-attachments/assets/bf5863e2-3267-44b3-ad65-825ecbc75054">
 
Ir tada paleidžiam ir gaunam tokį rezultatą:

<img width="624" alt="Screenshot 2024-11-13 at 23 47 44" src="https://github.com/user-attachments/assets/95f50488-d1ca-4afa-a20b-ea15bc5acf66">

### Xcode tvarkymas:
Einam į project->targets->Build Settings
Visų pirma reikia nurodyti Header Search Paths:

<img width="923" alt="Screenshot 2024-11-13 at 23 50 09" src="https://github.com/user-attachments/assets/022f318e-e3fb-493a-83ef-5d2f5d7c48dd">

Tada Library Search Paths:

<img width="754" alt="Screenshot 2024-11-13 at 23 50 43" src="https://github.com/user-attachments/assets/681739e7-2f19-450c-aeeb-2d5c3d4e8be3">

Tada einam Linking General -> Other Linker Flags
@ia nurodom tuos visus flag'us, kuriuos gavom iš terminalo:

<img width="903" alt="Screenshot 2024-11-13 at 23 52 25" src="https://github.com/user-attachments/assets/211ae566-e38b-46cb-a7da-df8adaa0d003">

Dar papildomai nurodžiau path'ą į pkconfig:

<img width="926" alt="Screenshot 2024-11-13 at 23 58 14" src="https://github.com/user-attachments/assets/a9b0ef8b-c79b-4224-b8a7-fbf56d800e0d">

Labai svarbu pakeisti C++ language dialect: 
<img width="953" alt="Screenshot 2024-11-14 at 00 04 38" src="https://github.com/user-attachments/assets/42e01ea1-7954-4f32-8155-66711b84fece">

Ir turėtų veikti, žinoma meta daug įspėjimų, bet veikia: 

<img width="1122" alt="Screenshot 2024-11-14 at 00 05 51" src="https://github.com/user-attachments/assets/1caf3e8a-c42e-4c54-8561-2a4a7ed169ec">

