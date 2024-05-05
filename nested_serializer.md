# Part 1:Nested Serializer
## models.py
```bash
# -*- coding: utf-8 -*-
from __future__ import unicode_literals

from django.db import models

# Create your models here.


class Musician(models.Model):
    first_name = models.CharField(max_length=50)
    last_name = models.CharField(max_length=50)
    instrument = models.CharField(max_length=100)

    def __unicode__(self):
        return self.first_name


class Album(models.Model):
    artist = models.ForeignKey(Musician, on_delete=models.CASCADE, related_name='album_musician', null=True, blank=True)
    name = models.CharField(max_length=100)
    release_date = models.DateField()
    num_stars = models.IntegerField()
```
## serializers.py 
```bash
from .models import *
from rest_framework import serializers, fields


class AlbumSerializer(serializers.ModelSerializer):

    class Meta:
        model = Album
        fields = ('id', 'artist', 'name', 'release_date', 'num_stars')


class MusicianSerializer(serializers.ModelSerializer):

    class Meta:
        model = Musician
        fields = ('id', 'first_name', 'last_name', 'instrument')
```
## views.py
```bash
# -*- coding: utf-8 -*-
from __future__ import unicode_literals

from django.shortcuts import render
from .models import *
from .serializers import *
from rest_framework import generics
# Create your views here.


class MusicianListView(generics.ListCreateAPIView):
    queryset = Musician.objects.all()
    serializer_class = MusicianSerializer


class MusicianView(generics.RetrieveUpdateDestroyAPIView):
    serializer_class = MusicianSerializer
    queryset = Musician.objects.all()


class AlbumListView(generics.ListCreateAPIView):
    queryset = Album.objects.all()
    serializer_class = AlbumSerializer


class AlbumView(generics.RetrieveUpdateDestroyAPIView):
    serializer_class = AlbumSerializer
    queryset = Album.objects.all()
```
## urls.py 
```bash
url(r'^api/musicians/$', MusicianListView.as_view()),
url(r'^api/musicians/(?P<pk>\d+)/$', MusicianView.as_view()),
url(r'^api/albums/$', AlbumListView.as_view()),
url(r'^api/albums/(?P<pk>\d+)/$', AlbumView.as_view()),
```
## MusicianListView
```bash
[
    {
        "id": 1,
        "first_name": "ganesh",
        "last_name": "xeno",
        "instrument": "Guitar"
    }
]
```
## AlbumsView
```bash
[
    {
        "id": 1,
        "artist": 1,
        "name": "NEO",
        "release_date": "2017-07-07",
        "num_stars": 4
    },
    {
        "id": 2,
        "artist": 1,
        "name": "NEO 2",
        "release_date": "2017-07-22",
        "num_stars": 4
    }
]
```
## MusicianSerializer
```bash
from .models import *
from rest_framework import serializers, fields


class AlbumSerializer(serializers.ModelSerializer):

    class Meta:
        model = Album
        fields = ('id', 'artist', 'name', 'release_date', 'num_stars')


class MusicianSerializer(serializers.ModelSerializer):
    album_musician = AlbumSerializer(read_only=True, many=True)

    class Meta:
        model = Musician
        fields = ('id', 'first_name', 'last_name', 'instrument', 'album_musician')
```
## MUsicianListView:
```bash
[
    {
        "id": 1,
        "first_name": "ganesh",
        "last_name": "xeno",
        "instrument": "Guitar",
        "album_musician": [
            {
                "id": 1,
                "artist": 1,
                "name": "NEO",
                "release_date": "2017-07-07",
                "num_stars": 4
            },
            {
                "id": 2,
                "artist": 1,
                "name": "NEO 2",
                "release_date": "2017-07-22",
                "num_stars": 4
            }
        ]
    }
```
# Part 2:
## Nested Serializer:
```bash
from .models import *
from rest_framework import serializers, fields


class AlbumSerializer(serializers.ModelSerializer):

    class Meta:
        model = Album
        fields = ('id', 'artist', 'name', 'release_date', 'num_stars')


class MusicianSerializer(serializers.ModelSerializer):
    album_musician = AlbumSerializer(many=True)

    class Meta:
        model = Musician
        fields = ('id', 'first_name', 'last_name', 'instrument', 'album_musician')

    def create(self, validated_data):
        albums_data = validated_data.pop('album_musician')
        musician = Musician.objects.create(**validated_data)
        for album_data in albums_data:
            Album.objects.create(artist=musician, **album_data)
        return musician

    def update(self, instance, validated_data):
        albums_data = validated_data.pop('album_musician')
        albums = (instance.album_musician).all()
        albums = list(albums)
        instance.first_name = validated_data.get('first_name', instance.first_name)
        instance.last_name = validated_data.get('last_name', instance.last_name)
        instance.instrument = validated_data.get('instrument', instance.instrument)
        instance.save()

        for album_data in albums_data:
            album = albums.pop(0)
            album.name = album_data.get('name', album.name)
            album.release_date = album_data.get('release_date', album.release_date)
            album.num_stars = album_data.get('num_stars', album.num_stars)
            album.save()
        return instance
```
## Output
```bash
# Post data to the MusicianListView to create.
{
    "first_name": "ganesh",
    "last_name": "xeno",
    "instrument": "Guitar",
    "album_musician": [
        {
            "name": "NEO",
            "release_date": "2017-07-07",
            "num_stars": 5
        },
        {
            "name": "NEO 2",
            "release_date": "2017-07-22",
            "num_stars": 4
        }
    ]
}

# PUT data to the MusicianListView to update data.
{
    "id": 1,
    "first_name": "ganesh",
    "last_name": "xeno",
    "instrument": "Guitar",
    "album_musician": [
        {
            "id": 1,
            "artist": 1,
            "name": "NEO",
            "release_date": "2017-07-07",
            "num_stars": 5
        },
        {
            "id": 2,
            "artist": 1,
            "name": "NEO 2",
            "release_date": "2017-07-22",
            "num_stars": 4
        }
    ]
}
```


