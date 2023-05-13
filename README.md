<h1 align="center">Aleo налаштування І</h1>

## Огляд

Для застосування SNARKs необхідно згенерувати певні параметри, щоб досягти високої ефективності (малі розміри доказу, швидкий час доказу та перевірки). Ці параметри генеруються за допомогою іншого набору параметрів, які повинні залишатися секретними. Ми називаємо ці секретні параметри "токсичним відходом".  Якщо доказувач знає ці секрети, [то він може генерувати достовірні докази для невірних тверджень](https://medium.com/qed-it/how-toxic-is-the-waste-in-a-zksnark-trusted-setup-9b250d59bdb4), порушуючи справедливість. Це небажано!

Для того, щоб гарантувати, що ніхто ніколи не дізнається про ці секрети, ми можемо генерувати їх розподіленим способом. Кожен учасник цієї так званої "церемонії" зробить свій внесок у генерацію параметрів зі своїм власним секретом. Якщо хоча б 1 учасник буде чесним і знищить свій секрет, то у зловмисника не буде можливості створити фальшиві докази.

Цей репозиторій містить впровадження для багатопартійних обчислень. [BGM17](https://eprint.iacr.org/2017/1050)
Церемонія поділяється на дві фази: одна генерує Powers of Tau, а інша "спеціалізує" їх на надану арифметичну схему для Groth16 SNARK.

Зауважте, що згенеровані Powers of Tau можуть бути повторно використані для будь-якої іншої фази 2 або для реалізації інших механізмів, таких як схема комітування поліномів [KZG10](https://www.iacr.org/archive/asiacrypt2010/6477178/6477178.pdf).

Щоб дізнатися, як забезпечити правильне виконання церемонії, дивіться файл [`RECOMMENDATIONS.md`](RECOMMENDATIONS.md).

## Налаштування I

### Фаза 1 (Powers of Tau)

1. A coordinator generates an accumulator
1. Participant downloads the latest accumulator
1. Participant contributes their randomness to the accumulator (randomness is permantently deleted after this step)
1. Participant uploads the accumulator back to the coordinator
1. Coordinator verifies the accumulator was transformed correctly and produces a new challenge

Важливим аспектом цієї процедури є те, що вона може ніколи не закінчуватися.
Це дозволяє SNARKs, які використовують KZG10, мати "неперервне" налаштування. 
Якщо учасник не довіряє налаштуванню, він може внести свій вклад у Powers of Tau та створити KZG10 з новими параметрами.

## Налаштування II

### Фаза 2 (Параметризація до Groth16)

1. Coordinator "prepares" the parameters from Phase 1 and converts them to Lagrange Coefficients
1. Participant downloads the latest state of the parameters
1. Participant contributes their randomness to the parameters (randomness is permantently deleted after this step)
1. Participant uploads the parameters back to the coordinator
1. Coordinator verifies the accumulator was transformed correctly
1. Loop from 2 for all participants

Це дає параметри, які можна використовувати для побудови SNARKs Groth16 для даної схеми.
Установка є коректною, якщо принаймні 1 учасник був чесним і знищив свій "токсичний залишок" на кроці 3.

## Посібник з будівництва

Побудуйте за допомогою команди `cargo build (--release)`. Ви отримаєте дві виконувані програми `phase1` та `prepare_phase2` у директорії  `target/` directory.

Протестуйте за допомогою команди `cargo test --all`.

Виконайте бенчмарки з допомогою команди `cargo bench --all` (використовує [`criterion`](https://github.com/bheisler/criterion.rs))

Якщо ви внесете зміни, не забудьте виконати `cargo fmt` та `cargo clippy --all-targets --all-features -- -D warnings`

Усі крейти вимагають Rust 2018 року і протестовані на наступних каналах:
- `1.39.0`
- `stable`

Якщо у вас немає встановленого Rust, виконайте команду: `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`

## Структура каталогу

Цей репозиторій містить декілька сховищ Rust, які реалізують різні будівельні блоки MPC. Високорівнева структура репозиторію виглядає наступним чином:
- [`phase1`](phase1):Rust скринька, яке забезпечує накопичувач для Powers of Tau. Він працює у багатопоточному режимі та працює "пакетами", що дозволяє обчислювати великі степені у середовищах з обмеженими ресурсами.
- [`phase2`](phase2):Rust-утиліта, яка надає обгортку над параметрами Groth16, яка також містить перевірену стенограму дотеперішніх внесків до фази спеціалізації
- [`setup1-contributor`](setup1-contributor): Rust скринька для учасника Aleo Setup I
- [`setup1-verifier`](setup1-verifier): для верифікатора Aleo Setup I
- [`setup2`](setup2): Rust скринька для запуску Aleo Setup II
- [`setup-utils`](setup-utils): Утиліти для спільного використання у різних пакунках, що стосуються вводу/виводу, математичних операцій та помилок.

## Ліцензія

Ця бібліотека є збіркою репозиторіїв, які ліцензовані за різними стандартними ліцензіями. Будь ласка, зверніться до кожного окремого репозиторію для отримання інформації про його ліцензію.
