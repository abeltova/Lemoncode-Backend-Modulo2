# Lemoncode-Backend-Modulo2

## **Introducción**

Aquí tienes el enunciado del modulo 2, creat un repo en Github, y añade un readme.md incluyendo enunciado y consulya (lo que pone aquí Pega aquí tu consulta)
## **Basico**
### **CRUD**

Crear una BBDD y hacer CRUD

#### **Crear Base de datos**

    use testcrud

#### **Crear Colección**

    db.createCollection('testCollection')

#### **Insertar un documento**

    db.testCollection.insertOne({
        name: 'Abel Tovar', 
        age: 36, 
        datebirth: new Date('1985-08-21')
    });

#### **Insertar varios documentos**

    db.testCollection.insertMany([
        { 
            name: 'Mortadelo Pi', 
            age: 57, 
            datebirth: new Date('1965-07-14') 
        }, 
        { 
            name: 'Filemón Pi', 
            age: 53, 
            datebirth: new Date('1968-01-14') 
        }
    ]);

#### **Actualizar un documento**

    db.testCollection.updateOne(
    {
        _id: ObjectId("623d938d900754148e2cea14")
    },
    {
        $set: {'name': "Mortadelo López"}
    });

#### **Eliminar un documento**

    db.testCollection.deleteOne(
    {
        _id: ObjectId("623d938d900754148e2cea14")
    });

### **Restaurar backup**

Vamos a restaurar el set de datos de mongo atlas airbnb.

Lo puedes encontrar en este enlace: https://drive.google.com/drive/folders/1gAtZZdrBKiKioJSZwnShXskaKk6H_gCJ?usp=sharing

Para restaurarlo puede seguir las instrucciones de este videopost: https://www.lemoncode.tv/curso/docker-y-mongodb/leccion/restaurando-backup-mongodb

> Acuerdate de mirar si en opt/app hay contenido de backups previos que tengas que borrar

### **General**

En este base de datos puedes encontrar un montón de apartementos y sus reviews, esto está sacado de hacer webscrapping.

**Pregunta** Si montarás un sitio real, ¿Qué posible problemas pontenciales les ves a como está almacenada la información?

    Aparte de que la información no está segmentada ni organizada en diferentes colecciones, al tener documentos tan grandes el rendimiento en lectura se verá seriamente afectado llegando en algunos casos a saturar el Working Set. 

### **Consultas**
#### **Básico**
- Saca en una consulta cuantos apartamentos hay en España.

        db.getCollection('listingsAndReviews').count({ 
            "address.country": {
                $regex: ".*Spain*."
            } 
        });

- Lista los 10 primeros:
    - Sólo muestra: nombre, camas, precio, government_area
    - Ordenados por precio.

            db.getCollection('listingsAndReviews')
            .find()
            .projection({
                _id: 0,
                name: 1,
                beds: 1,
                price: "$price",
                government_area: "$address.government_area"
            })
            .sort({
                price: 1
            })
            .limit(10); 

#### **Filtrando**
- Queremos viajar cómodos, somos 4 personas y queremos:
    - 4 camas.
    - Dos cuartos de baño.

            db.getCollection('listingsAndReviews')
            .find({
                beds: {$eq: 4},
                bathrooms: {$eq: 2}
            },
            {
                _id: 0,
                name: 1,
                beds: 1,
                bathrooms: 1,
            });

- Al requisito anterior,hay que añadir que nos gusta la tecnología queremos que el apartamento tenga wifi:

        db.getCollection('listingsAndReviews')
        .find({
            beds: {$eq: 4},
            bathrooms: {$eq: 2},
            amenities: {
                $all: ['Wifi']
            }
        },
        {
            _id: 0,
            name: 1,
            beds: 1,
            bathrooms: 1,
            amenities: 1
        });

- Y bueno, un amigo se ha unido que trae un perro, así que a la query anterior tenemos que buscar que permitan mascota Pets Allowed

        db.getCollection('listingsAndReviews')
        .find({
            beds: {$eq: 4},
            bathrooms: {$eq: 2},
            amenities: {
                $all: ['Wifi', 'Pets allowed']
            }
        },
        {
            _id: 0,
            name: 1,
            beds: 1,
            bathrooms: 1,
            amenities: 1
        });     

#### **Operadores Lógicos**
- Estamos entre ir a Barcelona o a Portugal, los dos destinos nos valen, peeero... queremos que el precio nos salga baratito (50 $), y que tenga buen rating de reviews

        db.getCollection('listingsAndReviews')
        .find({
            price: {$lte: 50},
            'review_scores.review_scores_rating': {$gte: 80},
            $or: [
            { 'address.country': {$eq: 'Portugal'} },
            { 'address.market': {$eq: 'Barcelona'} },
            ]
        },
        {
            _id: 0,
            name: 1,
            price: 1,
            review_scores: '$review_scores.review_scores_rating',
            country: '$address.country',
            city: '$address.market'
        });

### **Agregaciones**
#### **Básico**
- Queremos mostrar los pisos que hay en España, y los siguiente campos:
    - Nombre.
    - De que ciudad (no queremos mostrar un objeto, sólo el string con la ciudad)
    - El precio (no queremos mostrar un objeto, sólo el campo de precio)

            db.getCollection('listingsAndReviews')
            .aggregate([
                {
                    $match: {
                        'address.country': {
                            $eq: 'Spain'
                        }
                    }
                }, 
                {
                    $project: {
                        _id: 0,
                        name: 1,
                        city: '$address.market',
                        price: 1
                    }
                }
            ]);

- Queremos saber cuantos alojamientos hay disponibles por pais.
        
        db.getCollection('listingsAndReviews')
        .aggregate([
            {
                $group: {
                    _id: '$address.country',
                    cuantosAlojamientos: {
                        $sum: 1
                    }
                }
            }, 
            {
                $sort: {
                    _id: 1
                }
            }
        ]);

#### **Opcional**
- Queremos saber el precio medio de alquiler de airbnb en España.

        db.getCollection('listingsAndReviews')
        .aggregate([
            {
                $match: {
                    'address.country': {
                        $eq: 'Spain'
                    }
                }
            },
            {
                $group: {
                    _id: '$address.country',
                    priceAvg: {
                        $avg: '$price'
                    }
                }
            }, 
            {
                $project: {
                    _id: 0,
                    country: '$_id',
                    priceAvg: 1
                }
            }
        ]);


- ¿Y si quisiéramos hacer como el anterior, pero sacarlo por países?

        db.getCollection('listingsAndReviews')
        .aggregate([
            {
                $group: {
                    _id: '$address.country',
                    priceAvg: {
                        $avg: '$price'
                    }
                }
            }, 
            {
                $project: {
                    _id: 0,
                    country: '$_id',
                    priceAvg: 1
                }
            }
        ]);

- Repite los mismos pasos pero agrupando también por número de habitaciones:

        db.getCollection('listingsAndReviews')
        .aggregate([
            {
                $group: {
                    _id: {
                        country: '$address.country',
                        bedrooms: '$bedrooms'
                    },
                    priceAvg: {
                        $avg: '$price'
                    }
                }
            }, {
                $project: {
                    _id: 0,
                    country: '$_id.country',
                    bedrooms: '$_id.bedrooms',
                    priceAvg: 1
                }
            }, {
                $sort: {
                    country: 1,
                    bedrooms: 1
                }
            }
        ]);

#### **Desafío**
- Queremos mostrar el top 5 de apartamentos más caros en España, y sacar los siguentes campos:
    - Nombre.
    - Ciudad.
    - Amenities, pero en vez de un array, un string con todos los ammenities.

            db.getCollection('listingsAndReviews')
            .aggregate([
                {
                    $match: {
                        'address.country': { $eq: 'Spain' }
                    }
                }, 
                {
                    $sort: { price: -1 }
                }, 
                {
                    $project: {
                        _id: 0,
                        name: 1,
                        city: '$address.market',
                        amenities: {
                            $reduce: {
                                input: '$amenities',
                                initialValue: '',
                                'in': {
                                    $concat: [
                                        '$$value',
                                        '$$this',
                                        ',']
                                }
                            }
                        }
                    }
                }, 
                {
                    $limit: 5
                }
            ]);