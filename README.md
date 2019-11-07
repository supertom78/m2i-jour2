# m2i-jour2

J’aimerais vous parler de la différence entre tous ces mots que l’on peut lire un peu partout lorsque l’on commence à s’intéresser aux Tests. On peut confondre des termes et même penser que certains sont des synonymes. Sachez tout d’abord que tous, je dis bien TOUS ces mots, ne sont rien d’autre que des tests doubles.
Les termes utilisés dans cet article sont en anglais, bien que l’article soit en français; Je préfère utiliser “mocks”, plutôt que des mots comme « bouchons », terme trop générique et qui ne veut pas forcément dire grand chose.

Les tests doubles
Avant de rentrer dans le vif du sujet, on va d’abord voir ce que sont les tests doubles. Plutôt que de foncer tête baissée dans les tests à proprement parler, il est important de comprendre quelques définitions et leur intérêt. Du coup, je vais vous expliquer tout cela de force, contre votre gré, pour arriver à mes fins.

Tester une classe qui n’a aucune dépendance est généralement assez simple. La difficulté va commencer à se faire sentir (et cela devrait arriver presque tout le temps) lorsque les classes vont utiliser des dépendances.

Nous avons deux solutions pour tester une classe :

On peut tester la classe avec toutes ses dépendances, et si vous comptez faire ça, alors il serait bien que vous sortiez (Nan mais restez car le plus important c’est de tester le comportement de la classe).
Ou bien, nous pouvons essayer d’isoler notre classe de ses dépendances : C’est là où interviennent les tests doubles.
Ils ne sont rien d’autre que des duplications de classes qui sont plus coopératives que les vraies, qui vont faire ce qu’on leur demande gentiment et sans broncher. Un peu comme des doublures au cinéma, c’est le même physique, mêmes vêtements, mais lui, il va faire la cascade qu’on lui demande.

Il y a plusieurs moyens de créer des tests doubles, d’où ces mots tels que Mocks, Stubs, etc. Le problème c’est qu’ils sont généralement mal utilisés, et les personnes auront toujours tendance à utiliser le mot Mock alors qu’il s’agirait plus de Stub ou Fake.


Dummy
Pour faire la différence entre tous ceux-là je vais donc les expliquer un par un et commencer par le plus simple d’entre tous, le Dummy

Voici une interface :

interface UserInterface
{
    public function getPassword();

    public function getUsername();
}
Je l’implémente de cette manière :

class DummyUser implements UserInterface
{
    public function getPassword()
    {
        return null;
    }

    public function getUsername()
    {
        return null;
    }
}
Tada ! Voici un Dummy, mais au fait ça veut dire quoi ?

Dummy n’est rien d’autre qu’une classe dont on se fiche de comment elle est utilisée.

Un petit exemple :

class Invoice
{
    private $user;

    public function __construct(UserInterface $user)
    {
        $this->user = $user;
    }

    public function getPathToBinaryFile()
    {
        return __DIR__ . '/bin';
    }
}

$invoice = new Invoice(new DummyUser());
$invoice->getPathToBinaryFile();
Comme vous pouvez le voir ici, la méthode getPathToBinaryFile() n’utilise pas du tout $user. Du coup quand on voudra tester cette méthode, on peut lui donner un $user qui ne sert à rien vu qu’il n’est absolument pas utilisé.

Pour résumé, Dummy peut-être instancié sans aucune dépendance, vous n’avez pas besoin d’utiliser son implémentation et surtout si une méthode de DummyUser était appelée, cela devrait retourner une erreur.

D’ailleurs, d’ailleurs hein !

petite correction

Voici comment vous devriez faire votre classe DummyUser si vous voulez restreindre son fonctionnement :

cgit push --set-upstream origin prlass DummyUser implrments UserInterface
{
    public function getPassword()
    {
        $this->throwException();
    }

    public function getUsername()
    {
        $this->throwException();
    }

    private function throwException()
    {
        throw new Exception('NullPointerException');
    }
}
Au moins si une de vos méthodes est appelée par mégarde, PIM ! Vous serez prévenu

Stub
Alors pour les stubs, il s’agit d’implémenter une classe qui va répondre exactement ce que j’attends.

En voici un exemple :

class StubUser implements UserInterface
{
    public function getPassword()
    {
        return 'foo';
    }

    public function getUsername()
    {
        return 'Marvin';
    }
}
class SendEmail
{
    private $user;

    public function __construct(UserInterface $user)
    {
        $this->user = $user;
    }

    public function forgotPassword()
    {
        return sprintf(
            'Hi %s, your password is %s',
            $this->user->getUsername(),
            $this->user->getPassword()
        );
    }
}
$sendEmail = new SendEmail(new StubUser());

if ($sendEmail->forgotPassword() === 'Your password is foo') {
    echo 'Correct!';
} else {
    echo 'ohhhhh :-(';
}
Mais pourquoi est-ce que l’on ne met pas les valeurs souhaitées à StubUser via des setters par exemple ?

Et bien ce n’est pas le rôle dans ce test ! Vous aurez d’autres tests qui feront ces vérifications de changement de valeur. Ici vous souhaitez uniquement tester la méthode forgotPassword. Qui plus est cela vous crée un fort couplage qui ne sert à rien.

Ici votre test reste seul au monde.

https://www.youtube.com/watch?v=5AL8zU4VGp0

Si vos setters ne fonctionnent plus comme attendu, cela n’impactera pas ce test.

Fake
Fake est un peu comme un Stub, mais avec un peu de logique, une sorte de mini-implémentation de la vraie classe. Une bonne image serait un simulateur d’avion. Vous avez la vraie implémentation, le fait de piloter un avion, et son simulateur.

L’exemple le plus concret, en pratique, est celui de lire et d’écrire dans une base de données. Si vous ne l’avez jamais fait, il vous arrivera de vouloir tester que tout se passe bien avec votre base de données. Donc vous allez vouloir créer une base de données pour vos tests, il va aussi falloir créer des fixtures (insérer le jeu de données de tests), les lire et enfin les supprimer à la fin.
Vous vous en doutez, cela prend du temps, beaucoup de temps, même si cela parait ridicule au début, si vous arrivez à maintenir des tests partout sur votre projet, ce genre de test va vous prendre un temps précieux.
Si bien qu’à la fin vous ne lancerez simplement plus les tests, du fait du temps perdu, pour vérifier que tout fonctionne bien.

Voici un exemple d’usage :

interface PDOInterface
{
    public function findById($id);

    public function add($name);
}

class PDOFacade
{
    private $PDO;

    public function __construct(PDOInterface $PDO)
    {
        $this->PDO = $PDO;
    }

    public function findById($id)
    {
        return $this->PDO->findById($id);
    }

    public function add($name)
    {
        return $this->PDO->add($name);
    }
}


Et dans votre fichier de test :

$pdo = new PDOFacade(new DBPDO());
$pdo->add('Marvin');
$result = $pdo->findById(1);

if ($result['name'] === 'Marvin') {
    echo 'Youpiiii';
} else {
    echo 'oh noooo!';
}
Voici le code que l’on pourrait trouver dans votre implémentation concrète :

class DBPDO implements PDOInterface
{
    private $PDO;

    public function __construct()
    {
        $dsn = 'mysql:host=localhost'.
            ';dbname=my_database'.
            ';charset=utf8';
        $this->PDO = new PDO($dsn, 'login', 'password');
    }

    public function findById($id)
    {
        $this->PDO->beginTransaction();
        $query = $this->PDO->prepare(
            'SELECT * FROM my_table WHERE id = :id'
        );
        $query->execute(['id' => $id]);
        $response = $query->fetch();
        $query->closeCursor();

        return $response;
    }

    public function add($name)
    {
        $req = $this->PDO->prepare(
            'INSERT INTO my_table(name) VALUES(:name)'
        );
        $req->execute(array(
            'name' => $name,
        ));
    }
}
Comme vous pouvez le voir, nous avons mis en place une façade permettant d’effectuer tout ce que l’on souhaite sur notre base de données.

Maintenant, je souhaite pouvoir simuler la même chose sans avoir besoin de tester ma relation avec une base de données.

Pour cela, nous allons implémenter InMemoryPDO, qui va tout enregistrer dans un simple tableau.

class InMemoryPDO implements PDOInterface
{
    private $hashes;

    public function __construct()
    {
        $this->hashes = [];
    }

    public function findById($id)
    {
        return $this->hashes[$id - 1];
    }

    public function add($name)
    {
        $result = ['name' => $name];
        $this->hashes[] = $result;
    }
}
Ainsi, dans votre fichier de test, il ne vous reste plus qu’à modifier la première ligne en celle-ci :

$pdo = new PDOFacade(new InMemoryPDO());
Cela vous permet aussi de réduire considérablement le temps des tests.

Si vous faites du TDD/BDD, le mieux serait de commencer à créer un Stub et ensuite, si vous devez aller un peu plus loin, vous n’aurez qu’a faire une mise à jour du Stub en Fake.

Spy
Il reprend comme base Stub, donc on prend la même et on recommence !

Un Spy va avoir le même comportement qu’un Stub, mais il va nous permettre d’obtenir des informations supplémentaires, une fois le test effectué.

À nouveau un petit exemple ; prenons notre Stub de départ et prenons-le bien :

class SpyUser implements UserInterface
{
    public $getUsernameWasCalled = false;

    public function getPassword()
    {
        return 'foo';
    }

    public function getUsername()
    {
        $this->getUsernameWasCalled = true;
        return 'Marvin';
    }
}


Ainsi, à la fin de notre test, nous pouvons vérifier que celui-ci a bien été appelé.

$spyUser = new SpyUser();
$sendEmail = new SendEmail($spyUser);

if ($sendEmail->forgotPassword() === 'Your password is foo') {
    echo 'Correct!';
} else {
    echo 'ohhhhh :-(';
}

if ($spyUser->getUsernameWasCalled) {
    echo '$spyUser->getUsername() a été appelée!';
}


Comme vous vous en doutez (si si vous vous en doutez, mais vous ne le savez pas encore), c’est un exemple très simple. Un Spy n’est rien d’autre qu’un Stub qui enregistre des informations pendant le test que l’on pourra aller chercher par la suite. Imaginons que vous souhaitiez compter le nombre de fois où une méthode a été appelée, un Spy pourra vous être utile. Ou encore lorsque vos tests échouent et qu’il est difficile de savoir d’où vient le problème.

Mock
Ahhhh il est là, il est beau, celui que tout le monde attendait, le mot magique est lâché Mock.

Eh oui ! Ce mot que vous allez rapidement découvrir dans le monde des tests. Celui-ci est souvent utilisé à tort et à travers pour toutes les significations données ci-dessus.

Pour donner une définition concrète d’un Mock :

Il s’agit simplement d’un objet qui est une substitution complète de l’implémentation originale d’une classe concrète.

Donc pour créer un Mock d’une classe, on reprend les mêmes méthodes et on ajoute du code à l’intérieur pour vérifier son comportement. Et là on pourrait se dire qu’il s’agit d’un Stub. Eh bien non ! Car un Mock va plus loin que de simples informations écrites en dur. Un Mock peut aussi lever des exceptions s’il ne reçoit pas les bons appels. Il a aussi la possibilité de faire, en même temps, des vérifications pendant l’exécution du processus pour être sur que tout se passe comme prévu (un peu comme un Spy en fait !). Du coup, c’est généralement lui-même qui va s’occuper de faire les assertions. Vous n’avez donc plus à tester le Mock.

BOOM ! Voilà la dernière brique a été lâchée, des Mocks vont faire leurs propres tests pour savoir ce qu’ils testent (Inception !).

Voici donc la principale différence entre un Mock et un Stub ou un Fake : Il peut décider d’échouer.

Là où un Stub/Fake doit réussir car on effectue un test précis, un Mock peut, par exemple, s’il n’a pas les bons arguments pour une dépendance, décider d’échouer. Car il s’agit là bien là de l’utilité d’un Mock : une vérification comportementale. Comprenez qu’avec un Mock on ne cherche pas une valeur de retour, mais on s’intéresse plutôt à la méthode qui a été appelée, avec quels arguments ou encore combien de fois elle a été appelée.

Si je reprends l’exemple d’envoi de mail juste avant :

class MockUser implements UserInterface
{
    private $numberCalled = 0;
    private $numberShouldBeCalled = 0;

    public function getPassword()
    {
        return 'foo';
    }

    public function getUserName()
    {
        $this->numberCalled++;
        return 'Marvin';
    }

    public function setExpectedNumberCalls($number)
    {
        $this->numberShouldBeCalled = $number;
    }

    public function verify()
    {
        if ($this->numberShouldBeCalled != $this->numberCalled) {
            throw new Exception(sprintf(
                'Actual number of calls %d - expected %d.',
                $this->numberCalled,
                $this->numberShouldBeCalled
            ));
        }

        return true;
    }
}


Ainsi à la fin de notre test nous pouvons vérifier que celui-ci à bien été appelé.

$mockUser = new MockUser();
$mockUser->setExpectedNumberCalls(1);
$sendEmail = new SendEmail($mockUser);

if ($sendEmail->forgotPassword() === 'Your password is foo') {
    echo 'Correct!';
} else {
    echo 'ohhhhh';
}
$mockUser->verify();


Et voilà ! L’assertion se fait du côté du Mock. Sur l’exemple tout ira pour le mieux, mais changez le nombre attendu ou omettez carrément la ligne 2, et vous aurez une belle erreur.

Conclusion
Vous voilà enfin à la fin de cet article, toutes mes félicitations vous avez fait du bon boulot ! Vous allez enfin pouvoir commencer à écrire vos propres tests, génial .

Comme vous allez le constater, ce n’est pas de tout repos et c’est un peu long de devoir créer tout ce qu’il faut pour pouvoir mettre en place des tests. C’est pour combler ceci qu’il existe des outils de création de Mocks, comme PHPUnit, Prophecy, Codeception ou encore Mockery.

Ces outils vont vous servir à créer des Mocks à la volée, mais ça sera pour un prochain article .

À bientôt !
